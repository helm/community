# Event-Driven Architecture

This document describes an architectural transition inside of Helm. The client-server
model of Helm 2 will be replaced with a single event-driven model.

High level logic will _emit events_, while event responders will _handle events_.

Some event handlers will be internal-only. Other event handlers will be exposed
to the new lifecycle events subsystem (with its Lua engine). Those events will
be open to extension.

## The Lifecycle Events Architecture

Helm 3 is internally reorganized around an event model. Some events are
internal only, while others are exposed to the Lua scripting engine. In this
latter case, chart authors may script certain behaviors.

During various phases of an operation (like install), Helm will execute pre-defined
events. Charts will have an opportunity to respond to these events.

For example, during `helm install`, the process my look something like this:

1. Load chart
2. Merge values
3. Emit `pre-render` event
4. Render templates
5. Parse resulting manifest into objects
6. Emit `post-render` event
7. Validate all objects
8. Call `pre-install` event
9. Install the manifests into Kubernetes
10. Exit


#### Scripting Event Handlers

An embedded Lua interpreter can load scripts at runtime, which makes it
possible to write code which can be stored inside of a chart, but executed in
answer to a lifecycle event.

In Lua, executing a particular hook might look something like this:

```lua
  -- events is used to register event handlers.
  events.on("pre-render", 0.5, function (_)
      -- Accessing an object
      print("got chart " + _.chart.name)
      -- Mutating an object
      _.values.myvalue = "My Value"
  end)
```

The event handler format is: `events.on(_)`, where `_` is a variable pointing to
a context object (see next section).

### Script locations

Scripts are stored inside charts. They are stored in charts for the following
reasons:

- Charts remain self-contained. There is no need to negotiate on which plugins
  a chart needs.
- Dependency resolution is relegated to the chart scope
- The ordering problem is clearly solved (see below) if the scripts are located
  inside of the chart.


Lua scripts are stored in a chart's `ext/lua` folder. The `ext/lua/chart.lua` file
is loaded automatically along with the chart, and it may `require()` in other
Lua scripts.

The order of loading goes like this:

1. Chart is verified (if requested) against `.prov` file
2. Chart is loaded into memory
3. Chart.yaml is parsed, validated, and loaded
4. `ext/lua/chart.lua` files are parsed and loaded into a runtime
5. `chart-loaded` event is fired on chart that was loaded

> Note: Values from `-f` and `--set` are coalesced prior to chart loading.

Charts and subcharts may each have Lua. When events are triggered, the event
handler will do a depth-first execution of the `chart.lua` files, ending with the
topmost `chart.lua` file.

## The Context Object for Events

Every event will receive a context, which will contain data about that event. In pseudo-code, the context stucture looks like this:

```lua
context = {
  -- Loaded from chart.yaml
  chart = {
    version = function () return "v2" end,
    name = function () return "alpine" end,
    version = function () return "1.1.0" end,
    description = function() return "This is an example chart." end,
    keywords = function() return { "alpine", "utility", "debugging" } end,
    home = function() return "https://docs.helm.sh" end,
    sources = function() return { "https://github.com/helm/helm" } end,
    maintainers = function() return {
      name = "M Butcher",
      email = "mbutcher@example.com",
      url = "http://mbutcher.example.com"
    } end,
    icon = function() return "https://example.com/alpine.svg" end,
    appVersion = function() return "3.8" end,
    kubernetesVersions = function() return ">1.9.0" end,
  },
  -- All user-specified. None of these is hard-coded
  values = {
    rbac = { enabled = true },
    image = { name = "alpine", tag = "3.8"}
  },
  -- These are supplied by the system and the user
  release = {
    name = function() return "my-alpine" end
  },
  -- Information about the cluster that Helm is currently
  -- targeting
  capabilities = {
    helmVersion = function() return "3.0.0-alpha.1" end,
    -- Kubernetes version, as reported by Kubernetes
    kubernetesVersion = function() return "1.10.0" end,
    apiVersions = function() return {
      "v1",
      "batch/v1",
      "batch/v2",
      "batch/v2beta1",
      -- ...
    } end
  },
  -- Files specified in the chart, exclusing templates and
  -- the contents of ext/
  files = {
    { name = "config.json", data = "{}" }
  },
  -- Any gotopl files. Depending on the phase, modification of
  -- these may have no impact.
  templates = {
    { name = "pod.yaml", data = "apiVersion: v1\nkind: Pod\nname: {{.Release.Name }}\n#..." }
  },
  dependencies = {
    {name = "nginx", version = "1.2.0", repository = "https://example.com/charts"}
  },
  -- Depending on phase, this may contain the list of already
  -- rendered objects. It may be empty. Depending on the
  -- phase, the contents of this may, on return, be preserved.
  objects = {
    {
      apiVersion = "v1",
      kind = "Pod",
      name = "my-alpine"
    }
  }
}

```

Note that some of the properties in the context are functions.
These are implemented as functions because their values are
immutable. (The implementation of the function is informational pseudo-code, and is not the real implementation.)

> [name=Matt Butcher]
> Options for subcharts: We can either collapse them into one top-level context or nest subcharts in a `charts = { context::new() }` way.
> 


## Events

This is meant to be an exhaustive list of the events that will be emitted in Helm 3.

The following are events by command. If a command is not present, then that command
is not slated for inclusion in Helm 3.

For each command, the events are presented in the order that they occur.

### Create

Events for `helm create`:

- `pre-create`
- `post-create`

The `pre-create` event will receive a minimal context with no chart information. A `post-create` will receive the initialized chart. After a post-create, the chart will be written to the filesystem.

### Dependency Build

- `pre-dependency-build`
- `post-dependency-build`

The `pre-` hook will have access to the base chart's context.

The `pre-` hook will be able to modify the `dependencies` object before dependencies are resolved or fetched.

The `post-` hook will have access to the context as it appears after all dependencies have been resolved. The results of this hook are _not_ written to the filesystem.

### Dependency [List, Update]
No other `helm dependency *` commands have hooks

### Delete

- `pre-delete`

The `pre-delete` event will receive a minimal context. It will not receive the chart.

### Fetch

There are no defined hooks for this command

### Get [manifests, values]

There are no defined hooks for these commands

### History

There are no defined hooks for this command

### Home

There are no defined hooks for this command

### Init

There are no defined hooks for this command

### Inspect

There are no defined hooks for this command

### Install

- `pre-render`
- `post-render`
- `pre-install`

#### pre-/post-render

The pre- and post-render events are common across several operations.

`pre-render` occurs before the contents of the `context.templates` have been rendered. The `context.objects` array is thus empty prior to `pre-render`.

The `post-render` occurs after templates have been rendered to objects. Any further modifications to `context.templates` are ignored. The `context.objects` array will contain object representations of the things created after the template run.

#### pre-install

The `pre-install` event fires after `post-render`, and will receive the context returned from post-render. This event only fires on the install command, and thus is for install-specific work.

### Lint

- `pre-render`
- `post-render`
- `pre-lint`

The `pre-lint` event is fired after post-render, and is only fired for lint.

### Package

- `pre-package`

The `pre-package` hook is fired immediately before the package operation, and changes to context may be written to the packaged chart, but not onto the filesystem.

### Plugin [*]

There are no defined hooks for this command

### Repo [*]

There are no defined hooks for this command

### Rollback

_This is dependent on a possible re-architecting of `helm rollback` that will no longer include a render phase._

- `pre-render`
- `post-render`
- `pre-rollback`

The `pre-rollback` event fires after the post-render, and with the same context. This event only fires on rollback.

### Search

There are no defined hooks for this command

### Template

- `pre-render`
- `post-render`
- `post-template` (name may change)

The `post-template` event fires after post-render and receives the same context. It fires immediately before the output of the command is written to STDOUT.

### Test

- `pre-render`
- `post-render`
- `pre-test`
- `post-test`
- `failed-test`

`pre-test` is executed immediately before the test, and it may modify the charts prior to testing. `post-test` runs after the test. `failed-test` runs only if the test is marked failed.

### Upgrade

- `pre-render`
- `post-render`
- `pre-upgrade`
- `post-upgrade`

The `pre-upgrade` event fires after `post-render`, receiving the same context. `post-upgrade` fires after the chart has been submitted to the server. These two events only fire during upgrades.

### Verify

There are no defined hooks for this command

### Version

There are no defined hooks for this command
