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

The eventing model for Helm 3 can be represented by this pseudo-code (with
error handling elided):

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

There are two types of lifecycle events. Core events require exactly one
implementation. Optional events allow zero or more implementations. The lists
below are a non-exhaustive representative list.

#### Core (Exactly One Implementation)

These events are exposed only internally, and there will be no way to override
them in Helm 3.0.0. In the future, some of these may become exposed for
extension.

- render
- install
- upgrade
- delete
- list
- read-chart/write-chart
- validate

> If Go's type system is too rigid, core events may be implemented as pseudo-events,
> not using the same `emit()`/`on()` API pattern.

#### Optional (Zero or More Implementations)

These events will be accessible via the scripting interface.

- wait/watch
- ready
- pre-/post- for render, install, upgrade, delete, rollback, list, and validate
- chart-loaded

Support for features like alternative template engines is to be had by
scripting pre- or post-render hooks.

#### Scripting Event Handlers

An embedded Lua interpreter can load scripts at runtime, which makes it
possible to write code which can be stored inside of a chart, but executed in
answer to a lifecycle event.

In Lua, executing a particular hook might look something like this:

```lua
-- init is the entry point
function init(events) {
  -- events is used to register event handlers.
  events.on("pre-render", 0.5, function (name, namespace, chart, values) {
      -- Accessing an object
      print("got chart " + chart.name)
      -- Mutating an object
      values.myvalue = "My Value"
  })
}
```

The event handler format is: `events.on(eventName, weight, callback)`

Scripts are stored inside charts. They are stored in charts for the following
reasons:

- Charts remain self-contained. There is no need to negotiate on which plugins
  a chart needs.
- Dependency resolution is relegated to the chart scope
- The ordering problem is clearly solved (see below) if the scripts are located
  inside of the chart.


Lua scripts are stored in a chart's `ext/lua` folder, and are loaded along with
the chart. The order of loading goes like this:

1. Chart is verified (if requested) against `.prov` file
2. Chart is loaded into memory
3. Chart.yaml is parsed, validated, and loaded
4. `ext/lua` files are parsed and loaded into a runtime
5. Top-level chart's `init()` function is called.
6. `chart-loaded` event is fired on chart that was loaded

> Note: Values from `-f` and `--set` are coalesced prior to chart loading.

When multiple callbacks are specified for the same event, the callbacks will be
executed in weight-order (0 first, 1 last). If two have the same weight, their
execution order is nondeterministic.
