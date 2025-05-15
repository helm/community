---
hip: 9999
title: "Better Support for Resource Creation Sequencing"
authors: [ "Joe Beck <joebeck5705@gmail.com>", "Evans Mungai <mbuevans@gmail.com>" ]
created: "2024-11-15"
type: "feature"
status: "draft"
---

## Abstract

This HIP is to propose a new featureset in Chart v3, to provide chart authors ([Application Distributor](https://github.com/helm/community/blob/main/user-profiles.md#2-application-distributor) Helm user profile), a well supported way of defining what order chart resources and chart dependencies should be deployed to Kubernetes. Helm deploys all manifests at the same time. This HIP will propose a way for Helm to deploy manifests in batches, and inspect states of these resources to enforce sequencing.

At a high level, this HIP proposes the following
- Ability for chart authors to specify how to sequence deployment of **resources within a single chart**
- Ability for chart authors to specify how to sequence **subcharts within a parent chart**
- Ability for Helm operators and tool developers to enable sequencing behaviour using `--wait=ordered` CLI flag and `WaitStrategy=ordered` SDK parameter respectively

The HIP only targets resources deployed in the Helm install phase. Resources deployed as hooks are not sequenced using changes proposed here. Any sequencing of hooks will still rely on using `"helm.sh/hook-weight"` annotations. Annotations added to resources in hooks will be ignored.

## Motivation

The driving motivator here is to allow application distributors to control what order resources are bundled and sent to the K8s API server, referred to as resource sequencing for the rest of this HIP.

Today, to accomplish resource sequencing you have two options. The first is leveraging helm hooks, the second is building required sequencing into the application (via startup code or init containers). The existing hooks and weights can be tedious to build and maintain for application distributors, and built-in app sequencing can unecessarily increase complexity of a Helm application that needs to be maintained by application distributors. Helm as a package manager should be capable of enabling application distributors to properly sequence how their chart resources are deployed, and by providing such a mechanism to distributors, this will significantly improve the application distributor's experience.

Additionally, Helm currently doesn't provide a way to sequence when chart dependencies are deployed and this featureset would ideally address this.

## Rationale

This design was chosen due to simplicity and clarity of how it works for both the Helm developer and Application Operator

## Specification

At a high level, allow Chart Developers to assign named dependencies to both their Helm templated resources and Helm chart dependencies that Helm then uses to generate a deployment sequence at installation time.

For Helm CLI, the `--wait=ordered` flag will enable sequencing where resources are applied in groups of layers. SDK users will also be able to enable sequencing by setting a `Wait` boolean flag. Without this flag being enabled, resources will all be applied all at once which is the same behaviour in Chart v2.

Each release will store information of whether sequencing was used or not. This information is used when performing uninstalls and rollbacks.

The following annotations would be added to enable this.

*Additions to templates*
- `helm.sh/layer`: Annotation to declare a layer that a given resource belongs to. Any number of resources can belong to a layer. A resource can only belong to one layer.
- `helm.sh/depends-on/layers`: Annotation to declare layers that must exist and in a ready state before this resource can be deployed. Order of the layers has no significance

*Additions to Chart.yaml*
- `helm.sh/depends-on/subcharts`: Annotation added to `Chart.yaml` to declare chart dependencies, by name or tags, that must be deployed fully and in a ready state before the chart can be deployed. For the dependency or subchart chart to be declared ready, all of its resources, with their sequencing order taken into consideration, would need to be deployed and declared ready. Order of the charts has no significance.
- `depends-on`: A new field added to `Chart.yaml` `dependencies` fields that is meant to declare a list of subcharts, by name or tag, that need to be ready before the subchart in questions get installed. This will be used to create a dependency graph for subcharts.

The installation process would group resources in the same layer and send them to the K8s API Server in one bundle, and once all resources are "ready", the next layer would be installed. A layer would not be considered "ready" and the next layer installed until all resources in that layer are considered "ready". Readiness is described in a later section. A similar process would apply for upgrades. Uninstalls would function on the same layer order, but backwards, where a layer is not uninstalled until all layers that depend on it are first uninstalled. Upgrades would follow the same order as installation.

`helm.sh/layer` and `helm.sh/depends-on/layers` ordering annotations are only used when ordering resources in the same chart. When it comes to resources across charts one needs to declare `helm.sh/depends-on/subcharts` or `depends-on` in Chart.yaml to allow Helm to reason how to order install/upgrade/uninstall subcharts.

#### Template examples:
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
    helm.sh/depends-on/layers: ["database, queue"]
---
# resource 3
metadata:
  name: queue-processor
  annotations:
    helm.sh/layer: queue
    helm.sh/depends-on/layers: ["another-layer"]
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
  helm.sh/depends-on/subcharts: ["bar", "rabbitmq"]
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

In this example, Helm will first install and wait for all resources of `nginx` and `rabbitmq` dependencies to be "ready" before attempting to install `bar` resources. Once all resources of `bar` are "ready" then and only then will `foo` chart resources be installed. `foo` would require `rabbitmq` to be ready but since the subchart resources would have been installed before `bar`, this requirement would have been fulfilled.

This approach of building a directed acyclic graph (DAG) is prone to circular dependencies. During the templating phase, Helm will have logic to detect, and report any circular dependencies found in the chart templates. Helm will also provide a command to print the DAG for development and troubleshooting purposes.

### Readiness

In order to enforce sequencing, Helm will check that resources are ready using `kstatus` [ready condition](https://github.com/kubernetes-sigs/cli-utils/blob/master/pkg/kstatus/README.md#the-ready-condition) allowing helm to proceed installing the next group of resources, or fail the install. Users can override checks using optional `helm.sh/readiness-failure` and `helm.sh/readiness-success` annotations which are jsonpath queries to check status fields of the annotated resource. If any of the checks in the lists evaluate as true, the readiness check is determined as successful/failed. `helm.sh/readiness-timeout` annotation can be used to override how long Helm should wait when performing readiness checks before raising an error. It defaults to `10s`.

#### Example
```yaml
kind: Job
metadata:
  name: barz
  annotations:
    helm.sh/readiness-success: ["succeeded==1"] # status fields
    helm.sh/readiness-failure: ["failed==1"] # status fields
    helm.sh/readiness-timeout: 20s # timeout
status:
  succeeded: 1
```

A resources readiness is checked if there is a resource that depends on it as per the sequencing DAG.

#### Sequencing order

Resources with sequencing annotations in a chart would be deployed first followed by resources without. If the chart has a `helm.sh/depends-on/subcharts` annotation in the `Chart.yaml`, all resources of the defined subcharts would be deployed before deploying the main chart. If any sequencing annotations are defined in the subchart resources, Helm will enforce ordering of resources within. Sequencing of resources in a chart are sandboxed within the chart. Sequencing annotations will not affect resources in other charts.

- Installs: Helm will install resources in the order defined by the DAG. If any of the readiness checks fail or timeout, the entire install would fail and the release marked as failed. If `--atomic`, or its SDK equivalent is used, a rollback to the last successful install would take place.
- Uninstalls: Helm would uninstall resources in the reverse order they were installed, as per the sequencing order. The logic to delete each resource will not change.
- Rollbacks: Helm will check from the release object whether the revision being rolled back to, was installed in a sequenced manner. If it was, Helm will respect and enforce this order when installing resources from that revision. When deleting unneeded resources of the revision being rolled back from, the reverse order is followed just like uninstalls.
- `helm template` would print all resources in the order they would be deployed. Groups of resources in a layer would be delimited using a `## START layer: <chart> <layer-name>` comment indicating the beginning of each layer and `END layer: <chart> <layer-name>`.

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

Helm will continue to install/upgrade/uninstall/rollback all resources and dependencies at one go for all charts using `Charts v2` and below.

## Security implications

None.

## How to teach this

- Document how sequencing works in the official helm documentation website. Include ordering of Kubernetes resources that Helm enforces when applying resources to the cluster. Examples will be added to best demonstrate how this feature works.
- Document how this feature works for SDK users.

## Reference implementation

N/A

## Rejected ideas

1. A weight based system, similar to Helm hooks
    - Static numbering of the order is more challenging to develop and maintain
    - Modifying the order can lead to cascading changes.
    - Dynamically named system solves these problems for the application distributors.

## Open issues

## Prior raised issues

- https://github.com/helm/helm/pull/12541
- https://github.com/helm/helm/pull/9534
- https://github.com/helm/helm/issues/8439
- https://github.com/helm/community/pull/230
