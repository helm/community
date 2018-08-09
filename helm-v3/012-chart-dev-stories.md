# Chart v3 User Stories

This document contains user stories for Helm3 Charts. User stories provide guidance,
but are not necessarily a list of requirements.

All stories in this document are implicitly prefaced with "As a Chart Developer..."
and are of the form "As a _ROLE_ I want _TASK_ so that _GOAL_.


## General

- I want to re-use Helm 2 charts so that I don't have to rebuild the same thing
- I want to create charts the Helm 2 way so that I can continue developing
  the way I have been
- I do not want to have to understand all the details of every
  subchart I use so that I can compose complex applications quickly

## Extensions/Lua

These all assume a Lua engine, and describe things to be done in Lua scripts. See the current [events](002-events.md) and [chart structure](001-charts.md) proposals

### Values

- I want programmatic access to subchart values so that I can access defaults
- I want access to modify subchart values so that I can explicitly set a value
  for a subchart
- I want late access to a subchart's values so that I can discover any values
  that a subchart's scripts computed
- I want to use Lua to validate values before templates are rendered so that I
  can alert users to unexpected values before install fails
- I want to use Lua to modify values in subcharts programmatically so that I can
  expose a top-level set of values and computer subchart values
- I want to use Lua to alter subchart values so that I can read the values of one
  subchart and rewrite the values in another subchart (e.g. configuring a proxy
  based on the settings for a database or web server)
- I want to be able to intercept values in a top-level chart before _and_ after a
  subchart's event handler executes so that I can pre-seed data, let the subchart
  calculate results, and then retrieve and use those results. (e.g. SSL certificate
  generation)

### Templates
- I want to define template functions in Lua so that I can easily write custom
  template functions
- I want to render templates using a different template engine so that I can
  use my preferred template engine
  * Option A: Template engine implemented in pure Lua
  * Option B: Template engine invoked via `exec` call
- I want to intercept a template before it is rendered and be able to alter it so
  that I can dynamically replace parts of a template (example: regexp replace
  one template function/include with another)
- I want to replace a template in a subchart before it is rendred so that I can
  rewrite part of a chart dynamically

### Rendered Manifests

- I want to modify a generated manifest so that I can insert additional
  information (e.g. Istio sidecar)
- I want to delete a generated manifest prior to it being loaded so that I can
  prevent an object from being created (e.g. subchart creates a manifest I don't
  want)
- I want to add a new manifest post-render so that I can create an object based
  on the results of a render process (e.g. RBAC generation, service generation,
  storage class generation)

### Feature Discovery

- I want to query Kubernetes to see if a CRD exists so that I can decide whether
  to include a CRD definition
- I want to query whether a cluster supports RBAC so that I can decide whether
  to add RBAC rules
- I want to query for StorageClass so that I can pick an appropriate one for my
  PVC
- I want to look up DNS address (e.g. service discovery) so that I can find an
  endpoint, IP, or URL before installing/upgrading

### Events

- I want to declare a function that responds to an event (e.g. "render") so that
  I can modify a chart/values at a particular point in the lifecycle
- I want to allow subcharts to run their event handlers by default so that I don't
  have to read all of the subcharts to know what they do
- I want to be able to turn off a subchart's lifecycle event by name so that I can
  disable a particular event handler
- I want to turn off all event handlers for a chart so that I can prevent it from
  intercepting any events
- I want to have an opportunity to initialize my chart's code every time it is loaded
  so that my chart can be initialized into a known state consistently
- I want to be able to perform installation-specific actions before an install so
  that I can do custom install logic
- I want to be able to perform post-installation actions so that I can set up an
  application (or notify the user about an installation) after the app has been
  submitted to Kubernetes
- I want to be able to do upgrade-specific logic before the upgrade has been submitted
  to Kubernetes
- I want to be able to do post-upgrade logic so that I can set up an app or
  notify a user
- I want to do custom logic before an app is deleted so that I can prepare for
  a deletion
- I want to do custom logic after a deletion so that I can ensure that a teardown
  is successful (e.g. if no other apps are using a namespace, delete teh namespace;
  if no other apps are using a CRD, delete the CRD definition...)
- I want to be able to declare custom logic that determines whether an installed
  or upgrade application is considered _ready_.
- I want to be able to perform specific logic when an application is considered
  ready.

### Lua Libraries and Access

- I want to be able to break my Lua script into multiple files so that I can organize
  my code
- I want to be able to import code from subcharts so that I can re-use existing
  code
- I want to import existing pure Lua libraries that are stored in charts
- I want to use standard Lua libraries that do not have access to network or
  filesystem
- I want to optionally use network and filesystem libraries if approved by the
  user so that I can perform IO within a chart

## Library Charts

- I want to use charts as template definition libraries so that I can re-use
  common template code
- I want to use charts as containers for Lua libraries so that I can re-use
  Lua code

## Additional Considerations

- There are charts like OpenStack that have dozens of subcharts (and sub-subcharts)
