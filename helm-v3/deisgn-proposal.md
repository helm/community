# Helm 3 Design Proposal

This document contains a proposal for Helm 3. It assumes deep familiarity with Helm 2, and assumes that much of Helm 2's architecture will simply carry over without major change.

## Summary

- Tiller is gone, and there is only one functional component (`helm`)
- Charts are updated with a new Chart.yaml, libraries, schematized values, and the ext directory
- Helm will use a "lifecycle events" emitter/handler model.
- Helm has an embedded Lua engine for scripting some event handlers. Scripts are stored in charts.
- State is maintained with three CRDs: Application CRD, Release, and ReleaseVersion. Neither Helm CRD has a controller.
- Resources created by hooks will now be managed
- For pull-based DevOps workflow, a new Helm Controller project will be started
- Cross platform plugins in Lua that only have a runtime dependency on Helm
- A complementary command to `helm fetch` to push packages to a repository

## The Basic Architecture

Helm 3 is a single-service architecture. One executable is responsible for implementing Helm. There is no client/server split, nor is the core processing logic distributed among components.

The reference implementation of Helm 3 is a single command-line client with no in-cluster server or controller. This tool exposes command-line operations, and unilaterally handles the package management process.

The reference implementation has two distinct parts:

1. The command line façade, which translates commands, subcommands, flags, and arguments into a Helm operation
2. The Helm library, which provides the logic for executing all Helm operations.

By design, the Helm library _must_ be usable as a standalone library.

In Appendix A, we specify a second implementation of Helm 3 that provides a controller-oriented model with no client. The Helm Controller provides a CRD-based façade to receive commands, but executes them via the Helm library.

With this simplification, the following list comprises the components of Helm v3's core implementation:

* Charts
* Chart repositories
* `helm`, the CLI
* An extension mechanism (Lua scripts stored in the chart)
* A few in-cluster CRDs: Release, ReleaseVersion (Neither of which has an accompanying controller)

In this model, Tiller is removed and there are no operators or controllers that run in-cluster.

## Charts
The chart architecture remains largely unchanged with the exception that the `Chart.yaml`
file is now formatted as a Kubernetes resource.

The following changes are proposed to the existing chart format:

- Chart.yaml version 3
- The overrides/ directory
- The ext/ directory
- Schemata for values files

### Chart.yaml, Version 3

In an effort to mirror the construction of Kubernetes manifests, the new Chart.yaml file looks like this:

```yaml
apiVersion: helm.sh/v3
kind: Chart
metadata:
  name: myChart
  labels:
    heritage: helm
    version: 0.1.0
    appVersion: 3.6
data:
  description: Deploy a basic Alpine Linux pod
  home: https://github.com/kubernetes/helm
  sources:
    - https://github.com/kubernetes/helm
```

This file contains only metadata that describes the application (not the application instance), so multiple instances of the same chart can be installed.

The `heritage` label indicates which app created this. The CLI will use the heritage value of `helm`. Other components, such as an in-cluster controller, should specify their own value here.

Commonly accessed properties are moved into labels, while less frequently used or more finely structured data is relegated to the `data:` section. The table below illustrates the transitions from [Helm v2](https://github.com/kubernetes/helm/blob/master/docs/charts.md#the-chartyaml-file) to Helm 3:


| Helm 2 | Helm 3 | Notes |
| -------- | -------- | -------- |
| name     | metadata.name | |
| version | metadata.labels.version | |
| kubeVersion | data.kubeVersion | |
| keywords | data.keywords | Represented as a YAML list frabgment|
| description | data.description | |
| home     | data.home | |
| sources | data.sources | Represented as a YAML list fragment|
| maintainers | data.maintainers | following the same object syntax as in Helm 2|
| engine | REMOVED  | superseded by event system |
| icon | data.icon | |
| appVersion | metadata.labels.appVersion | Presumably, this is a field that should be queryable |
| deprecated | data.deprecated | |
| tillerVersion | REMOVED | There is no Tiller |

For the sake of completeness, we will provide a CRD defining this type.

*Note:* If the Application CRD proposal is workable during the course of early Helm 3 development, we will consider using that.


### The ext/ directory

The `ext/` directory is a reserved directory in v3 charts. It is reserved for placing extension scripts.

In Helm 3, Lua scripts are placed into `ext/lua/`. Examples of Lua scripts can be found below.

#### Sandboxing Model

When a chart is loaded, all of the scripts in `ext/lua` will also be loaded. At this time, a new runtime will be created for each chart. By default, the sandboxes will be prepared with a minimal set of built-in libraries. These libraries will not include network, os, or filesystem access.

Additional core libraries will be included if specified in the `ext/permissions.yaml` file.

```yaml
lua:
    - network
    - filesystem
```

The above permissions file asks that the Lua interpreter grant permission to the `network` and `filesystem` libraries.

> Note that the Lua engine will only make those libraries available if they are set in the permissions file.

When users install charts that ask for additional permissions, they will be prompted to allow:

```console
$ helm install stable/ishmael
Chart "ishmael" is requesting the following additional permissions:
  - network: Access the network
  - filesystem: Access the local filesystem
Allow? (y, yes, n, no) > 
```

An additional `-y`,`--yes` flag will allow global acceptance of permissions.

For fine-grained control, a `--accept-perms [strings]` may be provided to allow a
user to set a specific set of perms. If the chart asks for others, the install will
fail:

```
$ helm install stable/ishmael --accept-perms network,filesystem
```

The above will succeed if:

- The chart does not request any permissions
- The chart requires one or both of `network` and `filesystem`, but no other permissions

The above will fail if:

- The chart asks for any permissions other than `network` or `filesystem`


### Library Charts

Helm 3 supports a class of chart called a "library chart". This is a chart that is shared by other charts, but does not create any release artifacts of its own. A library chart's templates can only declare `define` elements. Globally scoped non-`define` content is simply ignored.

Library charts may also contain scripts (`ext/`). However, overlays are ignored.

Library charts are stored in `library/` in a Helm chart. Helm may hoist the library charts in subcharts into the parent chart.

Library charts are noted in the `library:` directive in the `requirements.yaml`.

```yaml
requirements:
  - name: apache
    version: 1.2.3
    repository: http://example.com/charts
  - name: mysql
    version: 3.2.1
    repository: http://another.example.com/charts
libraries:
  - name: common
    version: "^2.1.0"
    repository: http://another.example.com/charts
```

Libraries are semver ranged. When versions are processed, the highest global match is used. An incompatibility results in a failed chart render or build.

> [name=Matt Farina]
> How would library charts work with chart repositories? Or, maybe they aren't listed. If they are, is there a way to make them non-installable?
> [name=Matt Butcher]
> We could put a flag in the chart.yaml which would make such
> a chart uninstallable. I'm not sure, though, whether that's 
> necessary. Installing a library would be a no-op on Helm's
> side, so we could just have Helm output an error any time
> there are no objects to install (as Helm v2 does now).
> [name=Nikhil Manchanda]
> I would think that library charts would still be listed in repositories in case you wanted to fetch / inspect them. For usability, maybe having a flag that would not display libraries when searching would be a good idea? 
> 
> [name=Adnan Abdulhussein]
> I think we need a Chart.yaml field. Helm and other tools might want to differentiate between these in different ways. For example, does it make sense to include a non-library chart as a library in a parent chart? `helm search` should distinguish library charts and `helm install` should refuse to install a library chart. Other UIs like Kubeapps will also need to distinguish between library charts and normal charts.
> [name=Matt Butcher]
> I'm leaning Adnan's way... but if we do that, it will mean that you can't use a regular chart as a library chart ever. They will differe in kind, not merely in the way the requirements.yaml references them. Is that okay?
> 
> [name=Adnan Abdulhussein]
> I posed using a non-library chart as a library chart as a question, because I think it could be valid to use a non-library chart just for its helpers. Helm doesn't have to force dependencies marked as libraries in requirements.yaml to be marked as a library in Chart.yaml (it could produce a warning, though). So, I think a normal chart can be used as a library chart, but not the other way around. The main restriction being: you can't install a library chart.

### Schematized Values Files

In Helm 2, there is no schema that is applied to a values file.

For Helm 3, a [JSON schema](http://json-schema.org/specification.html) can be applied to a values file to validate it. The JSON schema file can be generated from the source values file (e.g. by introspecting the YAML), or it can be hand-authored.

For the sake of consistency, the JSON schema file will be stored as YAML data. The schema is stored in `values.schema.yaml`, parallel to `values.yaml`.

> [name=Adnan Abdulhussein]
> I'm wondering if there's a better way to do this that wouldn't mean having to maintain a separate file? Personally, I like the idea of annotating a values.yaml file with comments above a parameter to add schema, but that probably means implementing our own format rather than taking something like JSON schema.
> [name=Matt Fisher]
> I don't think there's a better way to do this unless we made values.yaml strongly typed.
> 
> https://github.com/kubernetes/helm/issues/2431 also goes into some thoughts around this proposal, which also agreed that a JSON schema approach would be best.
> 
> [name=Matt Butcher]
> Schema could be optional, in addition to being autogenerated by default.

For example, consider a simple `values.yaml` file:

```yaml
name: frontend
protocol: https
port: 443
```

This can be schematized into something like this:

```yaml
    title: Values
    type: object
    properties:
        name:
            description: Service name
            type: string
        protocol:
            type: string
        port:
            description: Port
            type: integer
            minimum: 0
        image:
            description: Container Image
            type: object
            properties:
                repo: 
                    type: string
                tag:
                    type: string
    required: 
        - protocol
        - port
```
> [name=Adnan Abdulhussein]
> Added an example to demonstrate what an object key would look like. IMO this is quite verbose. On the other hand, we could easily share schema for certain things (e.g. resources for Pods).

In many cases, the base types in a YAML file can be used to derive a base schema. Thus it may be desirable to automatically derive schemata if they are not supplied.

## The Lifecycle Events Architecture

Helm 3 is internally reorganized around an event model. Some events are internal only, while others are exposed to the Lua scripting engine. In this latter case, chart authors may script certain behaviors.

The eventing model for Helm 3 can be represented by this pseudo-code (with error handling elided):

```go

type InstallationRequest interface {
  // Chart represents the chart
  Chart() *Chart
  // Values returns the (coalesced) values supplied by the user
  Values() *Values
  // Config retruns the configuration, which may contain info from flag parsing
  // (like --dry-run or --debug)
  Config() *InstallConfig

  State() State
}

func Install(events EventHandler, r InstallationRequest) error {
  // Render Templates
  events.emit("pre-render", r)  // events.on("pre-render", name, namespace, chart, values)
  events.emit("render", r)
  events.emit("post-render", r) // events.on("post-render", name, namespace, chart, values, manifest)

  // Validate the results
  events.emit("pre-validate", r)
  events.emit("validate", r)
  events.emit("post-validate", r)

  // Install the chart
  events.emit("pre-install", r) // events.on("pre-install", name, namespace, chart, values, manifest)
  events.emit("install", r)
  events.emit("post-install", r)

  // Readiness: handle --wait if it is set to true
  if r.Config().Wait {
    events.emit("wait", r) // Do whatever waiting we have to do, and block
    events.emit("ready", r)
  }
}
```

The above is not intended to be the final API, but an illustration of how an
event-oriented model is represented.

### Lifecycle Event Types

There are two types of lifecycle events. Core events require exactly one implementation. Optional events allow zero or more implementations. The lists below are a non-exhaustive representative list.

#### Core (Exactly One Implementation)

These events are exposed only internally, and there will be no way to override them in Helm 3.0.0. In the future, some of these may become exposed for extension.

- render
- install
- upgrade
- delete
- list
- read-chart/write-chart
- validate

#### Optional (Zero or More Implementations)

These events will be accessible via the scripting interface.

- wait/watch
- ready
- pre-/post- for render, install, upgrade, delete, rollback, list, and validate
- chart-loaded

Support for features like alternative template engines is to be had by scripting pre- or post-render hooks.

#### Scripting Event Handlers

An embedded Lua interpreter can load scripts at runtime, which makes it possible to write code which can be stored inside of a chart, but executed in answer to a lifecycle event.

In Lua, executing a particular hook might look something like this:

```lua
events.on("pre-render", 0.5, function (name, namespace, chart, values) {
    -- Accessing an object
    println("got chart " + chart.name)
    -- Mutating an object
    values.myvalue = "My Value"
})
```

The event handler format is: `events.on(eventName, weight, callback)`

Scripts are stored inside charts. They are stored in charts for the following reasons:

- Charts remain self-contained. There is no need to negotiate on which plugins a chart needs.
- Dependency resolution is relegated to the chart scope
- The ordering problem is clearly solved (see below) if the scripts are located inside of the chart.


Lua scripts are stored in a chart's `ext/lua` folder, and are loaded along with the chart. The order of loading goes like this:

1. Chart is verified (if requested) against `.prov` file
2. Chart is loaded into memory
3. Chart.yaml is parsed, validated, and loaded
4. `ext/lua` files are parsed and loaded into a runtime
5. `chart-loaded` event is fired on chart that was loaded

> Note: Values from `-f` and `--set` are coalesced prior to chart loading.

Each chart that contains Lua scripts will have its own sandboxed interpreter. That is, if _Chart A_ contains _Subchart B_, each will have its own Lua sandbox, and they will not have access to each other's script objects.

When multiple callbacks are specified for the same event, the callbacks will be executed in weight-order (0 first, 1 last). If two have the same weight, their execution order is nondeterministic.

## State Storage

In Helm v3, state is tracked in-cluster by a trio of objects: The Application object, the Release object (one per chart install), and one or more ReleaseVersion objects, each of which represents a new version of a release. A `helm install` creates an Application, a Release, and a ReleaseVersion. A `helm upgrade` requires an existing Application and Release (which it may modify), and creates a new ReleaseVersion.

### Namespacing Changes

With Tiller gone, there is no reason to default to storing data in `kube-system`. Instead, Helm 3 will store release data _in the same namespace as the release's destination_.

Release and ReleaseVersions are stored **in the same namespace as the installed release itself**. In other words, if we install `nginx` into namespace `foo`, then both the Release and ReleaseVersion will also be installed into namespace `foo`.

For example, if no namespace is specified:

```console
$ helm install -n foo bar
# everything, including Release and ReleaseVersion is installed into default namespace
```

There is now only one releavant namespace, and the _tiller namespace_ goes away:

```console
$ helm install -n foo bar --namespace=dynamite
# installs release, releaseVersion, and un-namespaced charts into dynamite namespace.
```

As with Helm 2, if a resource explicitly declares its own namespace (e.g. with metadata.namespace=something), then Helm will install it into that namespace. But since the owner references do not hold over namespaces, any such resource will basically become unmanaged.
> [name=Taylor Thomas]
> This could be problematic for users. I know that at Nike we are using the cross-namespace functionality and we won't be the only ones
> 
> [name=Adnan Abdulhussein]
> Should we add the concept of a un-namespaced ClusterRelease which is created for charts that are cross-namespace?
> 
> [name=Matt Butcher]
> Maybe we should confirm whether or not owner references hold across namespaces. Cuz maybe this isn't actually an issue.

### The Application Object

To support common application interactions across different tooling, Kubernetes SIG Apps is developing a [common Application CRD](https://github.com/kubernetes-sigs/apps_application). This will enable Helm, Dashboard, kubectl, and others (e.g., ksonnet) to work together to some degree. For example, Helm can deploy an application that's visible within the Dashboard. Or, if Dashboard is used to delete an application all of the objects can be deleted for the application.

An Application will be deployed for each installation of a chart. The Application object is the owner of the Release object as represented by owner references.

> [name=Matt Farina]
> We still need to figure out if a Chart.yaml will be replaced with an Application CRD, if an Application CRD can be automatically managed based on a Chart.yaml file, or if both will be needed (hopefully not).
> 
> [name=Adnan Abdulhussein]
> I think this depends on how complicated the Application CRD will be. Based on the current KEP, I don't think this will be feasible (the Application spec keeps track of every installed resource).

### The Release Object

The release object contains information about a release, where a release is _a particular installation of a named chart and values._ This object describes the top-level metadata about a release.

At minimum, there are two necessary pieces of data a Release must track:

- The name of the release
- The curretly deployed version (ReleaseVersion) of this release

The release object persists for the duration of an application lifecycle, and is the owner of all ReleaseVersions, as well as of all objects that are directly created by the Helm chart. (These relationships may be represented by owner references.)

With this change, release names can now be scoped to namespace, instead of globally scoped as they were with Helm 2.

### The ReleaseVersion Object

The ReleaseVersion ties a release to a series of revisions (install, upgrades, rollbacks, delete). In Helm 2, revisions were merely incremental. Install created release v1, a subsequent upgrade went to v2, and so on. And in Helm 2, the Release and ReleaseVersion objects were collapsed into one.

For Helm 3, a Release has one or more ReleaseVersion objects associated with it. The Release object always describes the current head release. Each ReleaseVersion describes just one version of that release.

An upgrade operation, for example, will create a new ReleaseVersion, and then modify the Release object to point to this new version. Rollback operations can use older ReleaseVersion objects to roll back a release to a previous state.

A ReleaseVersion requires at least the following properties:

- name
- version
- values
> [name=Adnan Abdulhussein]
> There's a security concern to storing values, a lot of Charts use secrets directly from values today.

- chart or rendered manifest

> [name=Matt Butcher] Kubernetes cannot store an object larger than 1M, which numerous Helm users have identified as a problem.
> Instead of storing the complete chart tgz inside of the release version, we could...
> - Store just the rendered manifest
> - Store a reference to where the chart can be retrieved
>   - Chart repository URL...
>   - Or set up another storage mechanism
> - Not store anything, and remove the rollback feature
> Or we could just accept the 1M limit on data
 
> [name=Taylor Thomas]
> Definitely in favor of removing rollback. That functionality could be re-added with a plugin and would simplify core

> [name=Nikhil Manchanda]
> Perhaps, "rollback" can be implemented entirely on the client side as a rollforward -- upgrade to a new release version using the values from an older ReleaseVersion. 

> [name=Adnan Abdulhussein]
> +1 with removing rollback, I think that's better managed in version control or via some plugin.
> [name=Matt Butcher]
> Taking a look at `helm`, there are a number features that we lose if we don't store ReleaseVersion, such as `--reuse-values` all of `helm get`. How much are we okay losing between Helm 2 and Helm 3? I don't want to privilege architecture over usability.
> [name=Matt Fisher]
> Given that there's already a ton going on between Helm 2 and Helm 3 based on this proposal, I'd prefer holding off committing to removing functionality like `helm rollback` that has not been asked by users. Then again, that 1M limit on data *really* sucks in certain use cases. I don't have a pulse on which is the right path forward.
> [name=Matt Butcher]
> Doing a little testing today, it does appear that if we render a chart and store the rendered output, that gives us sufficient information to do a full rollback. If this is the case, then we can retain rollback pretty easily, though the ReleaseVersion will have secrets data stored in it. That said, it's also _much_ easier to lock down with RBAC.

#### Versions are ULIDs, not incremented integers
Helm 2 used incremented integers for version numbering. This is changing.

Version increments are ULIDs, not SemVer or integer counters. This is to prevent any race conditions when two revisions are pushed in near proximity. The version incrementor need not be negotiated with ULID; each new ULID is time-bound, but unique.


## Hook Annotations

In Helm 2, hook annotations could be placed on various manifests, and those annotations would result in special treatment.

In Helm 2, resources with hook annotations were unmanaged. In Helm 3, the owner reference is set on those, and they will be removed when the chart is deleted. There will be a way to override the management policy by setting an annotation for `helm.sh/hook-policy: unmanaged`
> [name=Taylor Thomas]
> Would hooks be needed anymore with the new eventing system? I think all we would need is something that can translate Helm v2 hooks into the new events behind the scenes

## Lua Plugins

The existing Helm plugin mechanism has enabled numerous extensions to Helm (e.g., [here](https://docs.helm.sh/related/#helm-plugins) and [here](https://github.com/search?q=topic%3Ahelm-plugin&type=Repositories)). While this has been useful there have been two shortcomings:

1. Many plugines are written as shell scripts assuming a POSIX environment and familiar linux-style userland. Helm, and Kubernetes as a whole, supports windows where these plugins don't work natively.
2. Other plugins are written in a compiled languages (e.g., Go) or a general scripting language (e.g., Python) that have system dependencies in order to work. These can have version complications as well (e.g., a system Python of 3 and a plugin Python need of 2.7).

Along with Helm supporting Lua for scripting chart extensions, it will support plugins in Lua. These plugins will only have a dependency on Helm to run and can be ported cross platform.
> [name=Matt Butcher]
> The main danger here is that we have to make it so that charts cannot have implicit reliance on Lua plugins. I think that is fairly straightforward, though. We just have to make sure that it's not possible to execute plugin code from within a chart.


### Plugin Sandboxing

Like charts, each plugin will be executed in an isolated Lua sandbox. Unlike chart scripts, plugins will be granted access to all core libraries, and will not require security prompts to accept.

## Pushing Charts To A Repository

Several registries have emerged with support for Helm charts including ChartMuseum and Quay (via Application Registries) along with extras built on top of other systems, such as object storage (e.g., see the [S3 Plugin](https://github.com/hypnoglow/helm-s3)). These registries are asking for and could benefit from a capability to push packages from the Helm client.

To support registries there will be a `helm push` command that works in a similar manner to `helm fetch`. It will support different systems via the URI/URL scheme and provide a default implementation for HTTP(S). Other systems, such as S3, can implement an uploader via a plugin for schemes Helm does not provide an implementation for.

> [name=Matt Farina] There very well may be additional work here around registry search and pluggable or otherwise extendable authenication mechanisms.
> 
> [name=Adnan Abdulhussein]
> What would the default HTTP(S) implementation be?
> [name=Matt Fisher]
> Adnan, would you mind clarifying your comment a little?
> 
> [name=Adnan Abdulhussein]
> Sorry, the proposal mentions there will be a default implemention for `helm push` to a regular HTTP(S) chart repository. It's unclear to me how this would actually work.
> 
> [name=Adnan Abdulhussein]
> Sounds like this is something that will be fleshed out more in Josh's proposal, so I'll wait for that :)

## Security Considerations

This architecture for Helm is more secure than Helm 2 (and equally secure to Helm Classic). The following security goals are met by this proposal:

- Helm now runs with the ID and permissions of the UserAccount or ServiceAccount executing the commands.
- RBAC can now be applied to each user
- On-the-wire security is as secure as the cluster is configured to be (e.g. if KUBECONFIG is set up for mTLS, that's what we use)
- Auditability is now a feature of the cluster, as each chart is "created by" the user who runs the request
- In-cluster CRDs can be secured with RBAC, too, providing access controls for releases.


## Appendix A: A Helm Controller

This section describes a second implementation of Helm. The goal of this implementation is to provide a controller/CRD façade for the core Helm features. _However, defining the specifics of this controller was deemed outside of the scope of this proposal, which focuses on the reference implementation of the Helm CLI_.

A client-only Helm follows a push-based model. To facilitate a pull-based model, we can also create a Helm controller that provides a CRD facade, but uses the Helm library.

It is suggested that this (a) be a core Helm project, but (b) is kept separately from the main Helm codebase.

This controller accepts Chart and values configuration as a new `HelmRequest` CRD, and then provides the following operations:

- Install
- Upgrade
- Delete
- Get
- List

Just like the client, the controller will operate on Release and ReleaseVersion objects. For that reason, the Helm CLI and the Helm CRD can be used in conjunction, and RBACs can even be applied to lock down certain features for one or the other. (For example, Helm CLI may be restricted to only read functions, while the controller can perform read and write operations)

> [name=Adnan Abdulhussein]
> How will the controller apply a user's RBAC permissions when installing a chart? Or is this not possible, and the controller will need to be restricted using service accounts?
> [name=Matt Fisher]
> It is not possible to apply a user's RBAC permissions when installing a chart via this model, which is why we decided to remove Tiller in Helm 3.

This implementation will necessarily have some limitations and restrictions that the client Helm does not; namely, it will not be able to handle large charts because there is a 1M limit to CRD size.

It is also assumed that the controller model may not choose to implement a number of the client features, such as dry runs, repository management, chart creation, etc.

## Appendix B: What Is a Package Manager?

If Helm is "the package manager for Kubernetes", we need to have a common explanation of what we mean by "package manager".

Helm's notion of package management is informed by _operating system package managers_ such as:
 - [APT](https://www.debian.org/doc/manuals/debian-reference/ch02.en.html)
 - [RPM](http://rpm.org/documentation.html)
 - [Homebrew](https://docs.brew.sh/)
 - [APK](https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management)
 - [Portage](https://wiki.gentoo.org/wiki/Portage)
 - [Snap](https://docs.snapcraft.io/core/)

It is important to not overzealously compare Helm to programming language library package managers, as such systems have a fundamentally different goal.

Package managers such as Apt, Homebrew, and RPM provide the following services:

- They define a package format
- They define a package repository architecture
- They define a security model (signing, verification, etc)
- They specify a programming langugage (or languages) to be used for management logic (e.g. shell scripts, Ruby)
- They manage the installation, upgrade, and deletion of a package
- They provide dependency resolution features and version management
- They expose configuration options that the user may override
- They start services (e.g. when installing a database, the database is started) and often provide tooling for administrators to manage those services (e.g. systemd/upstart/sysV init scripts)
- They provide tools to search, list, and discover packages
- They implement (or make it easy to implement) environment-specific (os, processor, distro) optimizations
- They provide an opinion as to how to best construct and maintain packages
- They surface documentation (typically via output during installation)

Package managers tend to involve all of these because it is important in package management to keep external dependencies to a minimum.

Consequently, if Helm is to fit into such a paradigm, it is natural that Helm will provide at least most, if not all of these.

Helm is not a configuration management tool (such as Chef or Puppet). Helm does provide certain features of such systems, though. Helm's templating and configuration model is a common feature of configuration managers (and also of package managers).

## Appendix C: Alternatives to Embedding Lua Engine

There were two main choices to be made about how to extend Helm with a better lifecycle event model:

1. Do we load lifecycle hooks as extensions, as part of a chart, or both?
2. What extension mechanism do we support?

In answer to the first question, we determined that the only sensible solution is to have the chart be the only way to implement lifecycle hooks. Loading via an external route (e.g. plugins) causes a new pain point: Charts are no longer compatible with all Helm installations. Instead, a chart would have dependencies on extensions. Which places Helm in the middle of a compatibility negotiation that is very difficult to solve without burdening the user.

In answer to the second question, we considered four alternatives before deciding on embedding a scripting engine. The following alternatives were considered as an alternative to embedding a scripting language. The pros/cons of embedding a scripting language leaned heavily in its favor.

### Option A: Implementing Lifecycle Hooks as Docker Images

This solution would cause each hook invocation to check whether registered charts (or perhaps plugins) declared handlers for a hook. Each handler would be executed as a Docker image, with the necessary data mounted into the image as a filesystem.

One way to declare lifecycle hooks is in the `Chart.yaml`:

```yaml
apiVersion: helm.sh/v3
kind: Chart
metadata:
  name: myChart
  labels:
    heritage: helm  # What app created this
    version: 0.1.0
    appVersion: 3.6
data:
  description: Deploy a basic Alpine Linux pod
  home: https://github.com/kubernetes/helm
  sources:
    - https://github.com/kubernetes/helm
  hooks:
    "pre-render":
     - name: "my-pre-render"
       image: alpine:3.6
       command: "echo hello"
```

- Do a Docker pull
- Do a Docker run
- Mount a volume with...
  - Chart
  - computed values
  - place to put artifacts
  - ???
- At image run completion...
  - If error, bubble the error
  - On success, read any data necessary out of the shared space and continue

PROS:
* Runnable image becomes the only format we need to support

CONS:
* User must have Docker installed and correctly configured
* Client needs access to the DockerHub registry, and possibly others
* Serialization and deserialization must be done from in-memory structures to filesystem entities
* It's unclear what changes on the FS will be picked up by the runtime. And this may vary with each hook

### Option B: Embed a Scripting Language

(This was the selected option, and is now in the propsal)

PROS:
* No additional tools needed to run
* Pattern is fairly straightforward for users to implement
* Performance is bettern than almost any other solution here

CONS:
* What the user can do is partially determined by what features we expose (e.g. file IO, network access, etc.)


### Option C: Re-implement Helm in Python|Ruby|TypeScript|JavaScript

This solution follows the route Heroku and others have gone: In order to make the core more extensible, rewrite it in a language that can dynamically load code.

The following languages have enough Kubernetes support that we could rewrite Helm in them:

- Python
- TypeScript/JavaScript
- (Possibly) Ruby

PROS:
- We can take advantage of rich existing ecosystems (like NPM)
- All of this languages are more popular than Go, so perhaps attracting contributors would be easier

CONS:
- We would have to rewrite Helm in its entirety
- Installation would be harder, since none of these has a static binary
- Some parts of Kubernetes (e.g. the apply logic) do not exist anywhere else but in the kubectl Go code
- It would likely become a maintenance nightmare to keep up with k8s

### Option D: Extend Plugins to Handle Lifecycle Events

In this solution, we would merely extend the existing Helm plugin model to allow it to declare and execute lifecycle hooks.

Plugins may declare which events they handle, and provide commands to run on an event. In YAML, this might be represented like this:

```yaml
apiVersion: helm.sh/v3
kind: Plugin
metadata:
  name: foo
data:
  hooks:
    "pre-render": "$PLUGIN pre-render"
    "validate": "$PLUGIN lint"
```

Each hook invocation would be executed against plugins that declare the hook. To accomplish this, Helm would "shell out" to each plugin.

PROS:
* Carries on an existing pattern
* Allows user to write in whatever language they want

CONS:
* Cross-platform becomes an issue
* Serialization and deserialization of all applicable data must be done for each plugin invocation
* It's hard to correlate a plugin to a chart (e.g. how do you programmatically say "this chart will only run if a plugin exists that does X with hook Y"?)

This appendix will be removed before the final proposal.

## Appendix D: Overlays (Removed)

An earlier draft included a provision for overlaying templates within charts. This section
was removed once we realized that the same thing can be easily accomplished using
the `post-render` event and some trivial Lua.

This appendix will be removed before the final proposal.
