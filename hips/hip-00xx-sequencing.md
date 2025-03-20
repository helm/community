---
hip: 9999
title: "Better Support for Resource Creation Sequencing"
authors: [ "Joe Beck <joebeck5705@gmail.com>", "Evans Mungai <mbuevans@gmail.com>" ]
created: "2024-11-15"
type: "feature"
status: "draft"
---

## Abstract

This HIP is to propose a new featureset in Helm 4 to provide application distributors, who create helm charts for applications, a well supported way of defining what order chart resources and chart dependencies should be deployed to Kubernetes. Helm deploys all manifests at the same time. This HIP will propose a way for Helm to deploy manifests in batches, and inspect states of these resources to enforce sequencing.


## Motivation

The driving motivator here is to allow application distributors to control what order resources are bundled and sent to the K8s API server, referred to as resource sequencing for the rest of this HIP.

Today, to accomplish resource sequencing you have two options. The first is leveraging helm hooks, the second is building required sequencing into the application (via startup code or init containers). The existing hooks and weights can be tedious to build and maintain for application distributors, and built-in app sequencing can unecessarily increase complexity of a Helm application that needs to be maintained by application distributors. Helm as a package manager should be capable of enabling application distributors to properly sequence how their chart resources are deployed, and by providing such a mechanism to distributors, this will significantly improve the application distributor's experience.

Additionally, Helm currently doesn't provide a way to sequence when chart dependencies are deployed and this featureset would ideally address this.

## Rationale

This design was chosen due to simplicity and clarity of how it works for both the Helm developer and Application Operator

## Specification

At a high level, allow Chart Developers to assign named dependencies to both their Helm templated resources and Helm chart dependencies that Helm then uses to generate a deployment sequence at installation time. The following annotations would be added to enable this.

*Additions to templates*
- `helm.sh/layer`: Annotation to declare a layer that a given resource belongs to. Any number of resources can belong to a layer. A resource can only belong to one layer.
- `helm.sh/depends-on/layers`: Annotation to declare layers that must exist and in a ready state before this resource can be deployed. Order of the layers has no significance

*Additions to Chart.yaml*
- `helm.sh/depends-on/charts`: Annotation added to `Chart.yaml` to declare chart dependencies, by name or tags, that must be deployed fully and in a ready state before the chart can be deployed. For the dependency or subchart chart to be declared ready, all of its resources, with their sequencing order taken into consideration, would need to be deployed and declared ready. Order of the charts has no significance.
- `depends-on`: A new field added to `Chart.yaml` `dependencies` fields that is meant to declare a list of subcharts, by name or tag, that need to be ready before the subchart in questions get installed. This will be used to create a dependency graph for subcharts.

The installation process would group resources in the same layer and send them to the K8s API Server in one bundle, and once all resources are "ready", the next layer would be installed. A layer would not be considered "ready" and the next layer installed until all resources in that layer are considered "ready". Readiness is described in a later section. A similar process would apply for upgrades. Uninstalls would function on the same layer order, but backwards, where a layer is not uninstalled until all layers that depend on it are first uninstalled. Upgrades would follow the same order as installation.

`helm.sh/layer` and `helm.sh/depends-on/layers` ordering annotations are only used when ordering resources in the same chart. When it comes to resources across charts one needs to declare `helm.sh/depends-on/charts` or `depends-on` in Chart.yaml to allow Helm to reason how to order install/upgrade/uninstall subcharts.

#### Templates example:
```yaml
# resource 1
metadata:
  name: db-service
  annotations:
    helm.sh/layer: database
---
# resource 2
metadata:
  name: my-app
  annotations:
    helm.sh/layer: app
    helm.sh/depends-on/layers: "database, queue"
---
# resource 3
metadata:
  name: queue-processor
  annotations:
    helm.sh/layer: queue
    helm.sh/depends-on/layers: another-layer
    helm.sh/depends-on/charts: "rabbitmq, minio"
```
In this example, Helm would be responsible for resolving the annotations on these three resources and deploy all resources in the following order. Resources in `database` and `queue` layers would be deployed at the same time. They would need to be ready before attempting to deploy `app` layer resources:

```
"database" layer:    [db-service]
                          ||
"queue" layer:            || [queue-processor]
                          ||       ||
                          ||       ||
                          \/       \/
                           \\     //
                            \\   //
                             \/ \/
"app" layer:                 [my-app]
```

#### Chart dependencies example

```yaml
name: foo
annotations:
  helm.sh/depends-on/charts: ["bar", "rabbitmq"]
dependencies:
  - name: nginx
    version: "18.3.1"
    repository: "oci://registry-1.docker.io/bitnamicharts"
  - name: rabbitmq
    version: "9.3.1"
    repository: "oci://registry-1.docker.io/bitnamicharts"
  - name: bar   # This is a subchart packaged with "foo" in charts dir. It's not pulled from a remote location
    version: "0.1.0"
    depends-on: ["nginx", "rabbitmq"]
    condition: bar.enabled
```

```
 [nginx] [rabbitmq]
    ||       ||
    ||       ||
    \/       \/
     \\     //
      \/   \/
       [bar]
        ||
        ||
        \/
       [foo]
```

In this example, helm will first install and wait for all resources of `nginx` and `rabbitmq` dependencies to be "ready" before attempting to install `bar` resources. Once all resources of `bar` are "ready" then and only then will `foo` chart resources be installed. `foo` would require `rabbitmq` to be ready but since the subchart resources would have been installed before `bar`, this requirement would have been fulfilled.

This approach of building a directed acyclic graph (DAG) is prone to circular dependencies. During the templating phase, Helm will have logic to detect, and report any circular dependencies found in the chart templates.

### Readiness

In order to enforce sequencing, a new `helm.sh/resource-ready` annotation would be used to determine when a resource is ready, allowing helm to proceed deploying the next group of resources, or failing a deployment. Helm would query, using jsonpath syntax, status fields of the annotated resource. Some native kubernetes resources that have stable APIs e.g `v1` resources such as `Pod` and `Deployment`, would have default queries which can be overriden. If Helm cannot determine how to check a resource's readiness when it should, it will do nothing and log this.

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

Resources with sequencing annotations would be deployed first followed by resources without. If the chart has a `helm.sh/depends-on/charts` annotation in `Chart.yaml`, all resources of the defined subcharts would be deployed and checked for readiness before deploying the chart. Only resources that Helm can determine their readiness for will be checked. Chart authors would need to annotate their chart resources accordingly. Helm will use default readiness checks for subcharts that do have their templates annotated.

`helm template` would print all resources in the order they would be deployed. Groups of resources in a layer would be delimited using a `## START layer: <chart> <name>` comment indicating the beginning of each layer and `END layer: <chart> <name>`.

```yaml
## START layer: foo layer1
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
    helm.sh/layer: layer1
## END layer: foo layer1
---
## START layer: bar layer1
# resource 3
metadata:
  name: fizz
  annotations:
    helm.sh/layer: layer2
    helm.sh/depends-on/layers: "layer1"
## END layer: bar layer2
```

## Backwards compatibility

Helm will continue to install/upgrade/uninstall all resources and dependencies at once, as adding the above defined annotations and depends-on field is when the installation behaviour would change. If a chart defines `helm.sh/depends-on/charts: ["subchart"]` annotation in `Chart.yaml`, Helm will wait for the subchart to be ready for all resources with default readiness checks. This will also apply to older subcharts that do not have any sequencing annotations.

## Security implications

None.

## How to teach this

TBD upon deciding on a design.
- Document how sequencing works in the official helm documentation website. Include ordering of kubernetes resources the Helm enforces when applying resources to the cluster.

## Reference implementation

N/A

## Rejected ideas

1. A weight based system, similar to Helm hooks
    - Static numbering of the order is more challenging to develop and maintain
    - Modifying the order can lead to cascading changes.
    - Dynamically named system solves these problems for the application distributors.

## Open issues

- Should this featureset take into account allowing application distributors to declare custom "readiness" definitions for given resources, besides the default?
- How will `--wait` and `--wait-for-jobs` work with sequencing annotations?
- Chart dependencies should be part of the Chart.yaml instead.

## Prior raised issues

- https://github.com/helm/helm/pull/12541
- https://github.com/helm/helm/pull/9534
- https://github.com/helm/helm/issues/8439
- https://github.com/helm/community/pull/230

N/A
