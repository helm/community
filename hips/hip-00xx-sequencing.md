---
hip: 9999
title: "Better Support for Resource Creation Sequencing"
authors: [ "Joe Beck <joebeck5705@gmail.com>" ]
created: "2024-11-15"
type: "feature"
status: "draft"
---

## Abstract

When an applicaiton requires dependencies to be spun up first before the application can start. Writing this sequencing into a helm chart today is very tedious using hooks and weights to sequence the resources, or some built-in application safety checks at runtime to achieve similar sequencing. I'd like to propose helm has a better sequencing mechanism that allows chart developers to specify dependency chains between resources, that doesn't leverage the current hooks and weights system.  

I'm not sure what exactly the best implementation of this would be, but my first naive approach would be a "depends on" system where a chart developer can specify something such as "this deployment `foo` depends on job `bar` completing before it is created". Then during installation/upgrades, helm would resolve the entire dependency chain and add resources to the cluster in the chain order.

## Motivation

Today, to accomplish any kind of sequencing you have two options. The first is leveraging hooks and weights, the second is building the sequencing into the application itself. 

Using hooks and weights can be a tedious process, as for example, running one pre-install job at weight 0, may require other resources such as config maps, secrets, and service accounts, to be created at weight "-1" to ensure the job can run. In a lot of cases, those same resources that exist at weight "-1" will need to be duplicated and exist outside of the context of the hook, which adds bloat to the chart installation. Additionally, for mor complex use cases, the weight system on hooks can become very tedious to maintain and get right on the chart developer side.

Building the sequencing into the application may reduce complexity in the chart itself, but it significantly increases complexity of the application when safety checks or the equivalent of `while(database not ready) { thread.sleep() }` need to be added just to satisfy the helm deployment use case, when that may be one of many ways to deploy an application.

By providing a mechanism to the chart developer that allows them to explicitly determine when resources are deployed to a cluster, we work around both of these issues, by reducing applicaiton complexity, and reducing the need to leverage hooks.

## Rationale

TBD upon deciding on a design.

## Specification

TBD upon deciding on a design.

## Backwards compatibility

TBD upon deciding on a design.

## Security implications

TBD upon deciding on a design.

## How to teach this

TBD upon deciding on a design.

## Reference implementation

N/A

## Rejected ideas

N/A

## Open issues

- The exact design of how Helm 4 should implement such a sequencing mechanism still needs to be determined.

## References

N/A