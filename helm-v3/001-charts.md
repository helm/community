# Charts


The chart architecture remains largely unchanged.

The following changes are proposed to the existing chart format:

- The ext/ directory and supporting requirements
- Library charts
- Schemata for values files

The following changes are proposed to the way charts will be referenced:

- The `--version` flag will be replaced by a new version reference system based on Docker

## The ext/ directory

The `ext/` directory is a reserved directory in v3 charts. It is reserved for
placing extension scripts.

In Helm 3, Lua scripts are placed into `ext/lua/`. Examples of Lua scripts can
be found below.

### Sandboxing Model

When a chart is loaded, the script `chart.lua` in `ext/lua` will also be loaded. All
subchart `ext/lua/chart.lua` scripts will be loaded into the same runtime, and then the
top-level (parent) chart's `init()` function will be executed.

With this construction, the parent chart will (a) have access to the objects
in the child charts, and (b) be responsible for initializing child charts. Library
support will be provided for simplifying this process:

```lua
function init(events) {
  -- Initialize subcharts
  subchart.init(events)

  -- Do other stuff
  event.on("pre-load", 0.5, function () {
    print("pre-load event")
  })
}
```

This design will make it possible for top-level charts to strategically overrride
the behavior of subcharts, or (as above) simply use them as-is.

By default, the chart sandbox will be prepared with a minimal set of built-in libraries.
These libraries will not include network, os, or filesystem access. The module
system (e.g. `require()`) will only allow in-chart libraries to be loaded.

Additional core libraries, such as os, network, and filesystem access, can be
loaded only of they are specified in the `ext/permissions.yaml` file.

```yaml
lua:
    - network
    - io
```

The above permissions file asks that the Lua interpreter grant permission to
the `network` and `io` libraries. This request will require user
confirmation.

When users install charts that ask for additional permissions, they will be
prompted to allow:

```console
$ helm install stable/ishmael
Chart "ishmael" is requesting the following additional permissions:
  - network: Access the network
  - io: Access the local filesystem
Allow? (y, yes, n, no) > 
```

An additional `-y`,`--yes` flag will allow global acceptance of permissions.

For fine-grained control, a `--accept-perms [strings]` may be provided to allow a
user to set a specific set of perms. If the chart asks for others, the install will
fail:

```
$ helm install stable/ishmael --accept-perms network,io
```

The above will succeed if:

- The chart does not request any permissions
- The chart requires one or both of `network` and `io`, but no other permissions

The above will fail if:

- The chart asks for any permissions other than `network` or `io`

When a chart contains subcharts, the presence of a `permissions.yaml` file will
result in a permissions prompt even if the subchart's Lua code is overridden.
This is because the overriding logic does not preclude the loading of the
libraries.

## Library Charts

Helm 3 supports a class of chart called a "library chart". This is a chart that
is shared by other charts, but does not create any release artifacts of its
own. A library chart's templates can only declare `define` elements. Globally
scoped non-`define` content is simply ignored.

Library charts may also contain scripts (`ext/`). However, overlays are
ignored.

Library charts are stored in `library/` in a Helm chart. Helm may hoist the
library charts in subcharts into the parent chart.

Library charts are noted in the `library:` directive in the
`requirements.yaml`.

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

Libraries are semver ranged. When versions are processed, the highest global
match is used. An incompatibility results in a failed chart render or build.

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

### Lua in Library Charts

Lua in library charts will be loaded as normal, and the `init()` functions will
be executed by `subcharts.init()` (if called). It is considered bad (but legal)
practice to include event handlers in library charts.

## Schematized Values Files

In Helm 2, there is no schema that is applied to a values file.

For Helm 3, a [JSON schema](http://json-schema.org/specification.html) can be
applied to a values file to validate it. The JSON schema file can be generated
from the source values file (e.g. by introspecting the YAML), or it can be
hand-authored.

For the sake of consistency, the JSON schema file will be stored as YAML data.
The schema is stored in `values.schema.yaml`, parallel to `values.yaml`.

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

In many cases, the base types in a YAML file can be used to derive a base
schema. Thus it may be desirable to automatically derive schemata if they are
not supplied.

## Version Reference System

In order to make things look and feel more like Docker, which many people have become comfortable with,
the `--version` flag will be removed on all subcommands (with the exception of `helm package`).

Referencing a specific chart version can be achieved using a new version reference system.
Very simply, a chart version can be specified using `NAME[:VERSION]`.

The following commands in Helm 2:

```
helm fetch stable/hackmd --version 0.1.0
helm search stable/hackmd --version 0.1.0
helm inspect stable/hackmd --version 0.1.0
helm install stable/hackmd --name hackmd --version 0.1.0
helm upgrade hackmd stable/hackmd --version 0.1.0
```

Will instead look like the following in Helm 3:

```
helm pull stable/hackmd:0.1.0
helm search stable/hackmd:0.1.0
helm show stable/hackmd:0.1.0
helm install hackmd stable/hackmd:0.1.0
helm upgrade hackmd stable/hackmd:0.1.0
```

If no version is specified, the latest stable semantically-versioned chart in the repository will be used by default
(same as it works in Helm 2).

Immutability, or "releases" as defined by
[App Registry](https://github.com/app-registry/appr/blob/master/Documentation/quick-start.md),
can be added at a later date without introducing any breaking changes by updating the version reference system to
accept additional characters (e.g. `@1.0.0`).

*Note: this change does not imply a change to the underlying repository system, only to the way versions
can be specified on the command-line.
Please see the [Repositories](./006-repositories.md) page for changes related to chart repositories.*
