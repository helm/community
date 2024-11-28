---
hip: 9999
title: "Better Support for Resource Creation Sequencing"
authors: [ "Joe Beck <joebeck5705@gmail.com>", "Evans Mungai <mbuevans@gmail.com>" ]
created: "2024-11-15"
type: "feature"
status: "draft"
---

## Abstract

This HIP is to propose a new featureset in Helm 4 to provide Helm developers a well supported way of defining what order chart resources and chart dependencies should be deployed to Kubernetes.

## Motivation

Today, to accomplish resource sequencing you have two options. The first is leveraging helm hooks, the second is building required sequencing into the application (via startup code or init containers). The existing hooks and weights can be a tedious to build and maintain for Helm developers, and built-in app sequencing can unecessarily increase complexity of a Helm application that needs to be maintained by Application Developers. Helm as a package manager should be capable of enabling developers to properly sequence how their applications are deployed, and by providing such a mechanism to developers, this will significantly improve the Helm and Application developer experiences.

Additionally, Helm currently doesn't provide a way to sequence when chart dependencies are deployed, and this featureset would ideally address this.

## Rationale

### Proposal 1: Annotations
This design was chosen due to simplicity and clarity of how it works for both the Helm developer and Application Operator

### Proposal 2: Weighted Resouces (without hooks)
This design was proposed as an alternative that functions very similarly to hooks today, to provide the same sequencing functionality outside of the scope of hooks.

## Specification

### Proposal 1: Annotations
At a high level, allow Chart Developers to assign annotations to both their Helm templated resources and Helm chart dependencies that Helm then uses to generate a deployment sequence at installation time. Three new annotations would be added to enable this described next.

- `helm.sh/layer`: Declare a layer that a given resource belongs to. Any numebr of resources can belong to a layer.
- `helm.sh/depends-on/layer`: Declare a layer that must exist and in a ready state before this resource can be deployed.
- `helm.sh/depends-on/chart`: Declare a chart dependency that must exist and in a ready state before this resource can be deployed. For the chart to be declared ready, all of its resources, with their sequencing order taken into consideration, would need to be deployed and declared ready.

A layer would not be considered "ready" and the next layer installed until:
- all jobs in the layer are DONE
- all other resources in the layer are READY

#### Example:
```yaml
# resource 1
metadata:
  name: foo
  annotations:
    helm.sh/layer: layer1
---
# resource 2
metadata:
  name: bar
  annotations:
    helm.sh/layer: layer2
    helm.sh/dependson/layer: layer1
---
# resource 3
metadata:
  name: fizz
  annotations:
    helm.sh/layer: layer3
    helm.sh/dependson/layer: layer1
    helm.sh/dependson/chart: someChartDependency
```
In this example, Helm would be responsible for resolving the annotations on these three resources and deploy all resources in the following order:

```
layer1:    [foo]
             ||
             ||   [someChartDependency]
             ||     ||
             \/     ||
layer2:    [bar]    ||
              \\   //
               \/ \/
layer3:       [fizz]
```

This approach is prone to circular dependencies. During the templating phase, Helm will have logic to detect, and report any circular dependencies found in the chart templates.

### Proposal 2: Weighted Resouces (without hooks)

Expand on the the weight capabilities of hooks, but allow them to be defined outside of the context of hooks for the chart. Add a new annotation `helm.sh/weight` that helm then uses to order when resources are deployed based on the same weighting rules that hooks use today.

#### Example:
```yaml
# resource 1
metadata:
  name: foo
  annotations:
    helm.sh/weight: "-2"
---
# resource 2
metadata:
  name: bar
  annotations:
    helm.sh/weight: "-1"
---
# resource 3
metadata:
  name: fizz
  annotations:
    helm.sh/weight: 0
```
In this example, Helm would be responsible for resolving the annotations on these three resources and deploy all resources in the following order:

```
-2:    [foo]
        ||
        \/
-1:    [bar]
        ||
        \/
0:     [fizz]
```

### Readiness

In order to enforce sequencing, new `helm.sh/resource-ready` annotation would be used to determine when a resource is ready, allowing helm to proceed deploying the next group of resources, or failing a deployment. Helm would query, using jsonpath syntax, status fields of the annotated resource. Some native kubernetes resources that have stable APIs e.g `v1` resources such as `Pod` and `Deployment`, would have default queries which can be overriden. In Helm cannot determine how to check a resource's readiness when it should, it will do nothing and log this.

#### Example
```yaml
kind: Job
metadata:
  name: barz
  annotations:
    helm.sh/resource-ready: "status.succeeded==1"
status:
  succeeded: 1
```

#### Sequencing order

Resources with sequencing annotations would be deployed first followed by resources without. If a resource has a `helm.sh/depends-on/chart` annotation, all resources of the referred subchart would be deployed and checked for readiness before deploying the dependant resource. Only resources that Helm can determine readiness for will be checked. Chart authors would need to annotate their chart resources accordingly.

Helm would scope each subchart layer annotation names using a delimiter such as `.` e.g `mysubchart.mylayer` to avoid any name collisions. This is an internal implementation detail rather than feature chart authors or operators would need to know.

`helm template` would pring all the resources in the order they would be deployed. The groups would be delimited using a `## Layer: <name>` comment indicate the beginning of each layer

```yaml
## Layer: layer1
# resource 1
metadata:
  name: foo
  annotations:
    helm.sh/weight: "-2"
---
# resource 2
metadata:
  name: bar
  annotations:
    helm.sh/weight: "-1"
---
## Layer: layer2
# resource 3
metadata:
  name: fizz
  annotations:
    helm.sh/weight: 0

```

## Backwards compatibility

Helm will continue to deploy all resources and dependencies at once, as adding the above defined annotations is when the installation behavior would change.

## Security implications

None.

## How to teach this

TBD upon deciding on a design.

## Reference implementation

N/A

## Rejected ideas

N/A

## Open issues

- Choose between Proposal 1 and Proposal 2
- Should this featureset take into account allowing Helm Developers to declare custom "readiness" definitions for a given layer, besides the default?

## References

N/A
