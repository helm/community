---
hip: "9999"
title: "Helm CRD updating"
authors: [ "George Jenkins <gjenkins@gmail.com>" ]
created: "2024-12-19"
type: "feature"
status: "draft"
helm-version: "4"
---

## Abstract

<!--
A short (~200 word) description of the technical issue being addressed.
-->

Helm has historically taken a conservative approach to Custom Resource Definitions (CRDs).
The concerns detailed in [HIP-0011](./hip-0011.md).

This HIP aims to improve Helm's support for CRD management, while remaining in line with the rationales in `HIP-0011`.
Ensuring chart consumers can continue to safely operate charts that provide CRDs.
But extending Helm's CRD management to enable updates to CRDs.


## Motivation

<!--
Clearly explain why the existing design is inadequate to address the problem
that the HIP solves.
-->

This HIP aims to improve the experience for chart authors and operators by extending Helm's ability to manage CRDs.
Helm's conservative approach to date has limited the abilty for Helm's users (both authors and chart operators) to package and use Kubernetes applications that make use of CRDs.
Today, Helm will only install CRDs (from the `crds/` directory) when installing a chart.
CRDs are otherwise ignored for additional operations, including upgrades.

CRDs have become more commonly utilized to extend Kubernetes functionality beyond its core primatives.
Being conservative ensured Helm erred on the side of safety and didn't create an irreversible position for Helm nor its users.
But has limited Helm's users, both chart operators and authors, to produce charts that expect to be able to manage CRDs.
As such, improvements to Helm's CRD management, that allows Helm and charts to better manage and update CRDs are desired.

The solution here aims to extend Helm's CRD management, while adhering to the concerns detailed in [HIP-0011](./hip-0011.md).
While providing for better CRD management, but continuing to provide for safe operation of charts by chart operators.


## Rationale

<!--
Describe why particular design decisions were made.
-->
The two main reasons inferred from [HIP-0011](./hip-0011.md) that Helm is conservative with CRDs are:
1. CRDs are a cluster-wide resource, and cluster-wide resources can more easily affect all users of a cluster.
   Rather than just the application / chart the operator is deploying, as operators and applications are often scoped to a single namespace.
   Frequently user That may affect multiple users within the cluster.
2. Custom Resources (CRs) can be used as a data store.
   And removing the CRDs for those will irrevocably remove any CRs and cause parmenat loss of any contained data

For 1., it is thought that Helm should not treat CRDs specially here.
Helm will readily operate on many other cluster wide resources today: Cluster roles, priority classes, namespaces, etc.
That the modification/removal of could easily cause breakage outside of the chart's release.

In general, Helm as a package manager should not try to prempt unintended functional changes from a chart.
Validating functional changes is well beyond the scope of a package manager.
Helm should treat CRDs the same as other (cluster-wide) resources, where a Helm chart upgrade that causes unintended functional effects should be reverted (rolled back) by the user (chart operator).
And as long as that rollback path exists, this is a suitable path for users to mitigate breaking functional changes.

For 2., Data loss should be treated more carefully.
As data loss can be irrevocable or require significant effort to restore.
And especially, an application/chart operator should not expect a chart upgrade to cause data loss.
Helm can prevent Data loss can be prevented by ensuring CRDs are "appended" only (with special exceptions in the specification below). An in particular, appending allows a rollback to (effectively) restore existing cluster state.


## Specification

<!--
Describe the syntax and semantics of any new feature.
-->

_Note: This specification applies to CRDs in a chart's `crds/` directory only. CRDs deployed via `templates/` are out of scope. As Helm only considered CRDs released via the `crds/` directory to be managed by Helm_

When installing for upgrading, Helm will install new CRDs from the `crds/` directory if they don't exist in the cluster.
If the CRD already exists, Helm will "append" or merge CRD updates into the cluster's existing CRD object with the logic detailed below.

When rolling back, Helm will simply revert merge in the previously installed version. Effectively, this is reverting to the prior "storage" version only.

When uninstalling: Helm will leave CRDs as is

The append/merge logic is as follows. When updsting an existing CRD resource, Helm will:

- merge/update CRDs metadata (labels, annotations etc)
- merge/update `/spec/names`
- append new versions to the `/versions` list
- update existing versions with the exception of the schema field
    - If in the future, functionality exists that shows a CRD version schema changes are backwards compatible, Helm may allow updating CRD version's schema field
    - In particular, Helm will expect the update from the chart to correctly set a single version to `storage: true`
- merge/update `/conversion` field
- set `/preserveUnknownFields` if the incoming or existing preserveUnknownFields values is set (logical OR)

If in the future, Kubernetes introduces new fields to the CRD API, Helm will ignore these fields by default.
Helm will consider introducing new logic to update any new new field that follows the rationale section as appropriate.

If the `--skip-crds` flag is supplied on install/upgrade/rollback, similar to today, Helm will ignore CRD changes.

## Backwards compatibility

The adjustment in CRD management behavior will be visible to end users.
Chart upgrades which previously ignored CRD changes will now action them.
Which may cause user facing behaviour

These behavioural changes are consider to acceptable for Helm 4.

## Security implications

None determined.

## How to teach this

Helm's CRD handling docs need to be updated: <https://helm.sh/docs/chart_best_practices/custom_resource_definitions/>

In particular, the merge logic described will need to be documented/described.

## Reference implementation

<!--
Link to any existing implementation and details about its state, e.g.
proof-of-concept.
-->

## Rejected ideas

<!--
Why certain ideas that were brought while discussing this HIP were not
ultimately pursued.
-->

### Deleting CRDs / versions

In order to prevent data loss, this HIP just considered how to update CRDs.
And did not consider how to remove CRD versions nor CRDs themselves.

## Open issues

<!--
Any points that are still being decided/discussed.
-->

## References

<!--
A collection of URLs or materials used as references through the HIP.
-->

- [HIP-0011: CRD Handling in Helm 3](./hip-0011.md)
- Kubernetes CRD docuemntation:
  - <https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/>
  - <https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/>
  - <https://kubernetes.io/docs/reference/kubernetes-api/extend-resources/custom-resource-definition-v1/>
