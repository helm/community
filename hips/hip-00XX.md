---
hip: 9999
title: "Utilize Server Side Apply for installs/upgrades"
authors: [ "George Jenkins <gvjenkins@gmail.com>" ]
created: "2023-07-07"
type: "feature"
status: "draft"
helm-version: "4"
---

## Abstract

<!-- A short (~200 word) description of the technical issue being addressed. -->
This HIP proposes for Helm to support Kubernetes' [Server-Side Apply](https://kubernetes.io/docs/reference/using-api/server-side-apply/) (SSA) for [object](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-object) management.

Helm 3 introduced the [three-way strategic merge and patch](https://helm.sh/docs/faq/changes_since_helm2/#improved-upgrade-strategy-3-way-strategic-merge-patches).
Allowing Helm to update objects which have been modified by other processes.
Kubernetes now offers a similar server-side process that has several advantages over the client-side apply (CSA) methods that Helm and kubectl (for example) have traditionally utilized.

<!--
The scope of this HIP is limited to incorporating SSA as an opt-in feature only. Deferring changing user's existing usage of the (client-side) three-way strategic merge patch processes (without opt-in). And considers how to deal with field conflicts and other compatibility concerns that may arise with the differences between SSA and CSA.
-->

## Motivation

<!-- Clearly explain why the existing design is inadequate to address the problem
that the HIP solves. -->

The primary motivation for Helm to switch to Server-Side Apply is to gain the benefits of Kubernetes server-side management of object upgrades.
The following describes the advantages of SSA from Kubernetes perspective, where SSA generally is considered to be superior:

https://kubernetes.io/blog/2022/10/20/advanced-server-side-apply/

Helm supporting SSA will allow Helm users to realize the benefits of SSA.
With Helm 4 utilizing SSA by default ([similar to kubectl](https://github.com/kubernetes/enhancements/tree/master/keps/sig-cli/3805-ssa-default)).

From a code maintenance benefit, Helm could eventually drop support for the strategic-merge patch CSA implementation/code.

## Rationale

<!-- Describe why particular design decisions were made. -->

With Helm 4, it was decided to opt-in to Helm utilizing SSA by default.
Allowing chart operators to gain the benefit of SSA without needing to know about the mechanism.
If during Helm 4 release candidate field usage, SSA as default is deemed to be unsuitable, Helm will revert to utilizing CSA by default.

A CLI flag will allow users to override SSA opt-in (revert to utilizing CSA), should a chart operator need to for a particular circumstance.

A chart release mechanism will allow Helm to track and follow SSA enablement for a given release. Tracking SSA enablement will allow Helm to automatically follow SSA enablement for a given release, and in revert to CSA (or SSA) during rollback if that prior release utilized CSA (or SSA).

Additionally it was desirable to mimic `kubectl`'s CLI interface/flags.
As a way for Helm to be consistent with an existing `kubectl` functionality.

## Specification

<!-- Describe the syntax and semantics of any new feature. -->

### General

Helm will add a new flag to the install, upgrade and rollback commands (modeled after the like `kubectl apply` flag).
And add a corresponding field to the respective install/upgrade/rollback SDK API objects:

`--server-side=false|true|auto`

Helm will default to `true` for installs, enabling SSA.
And `auto` for upgrade and rollbacks, where Helm will enable SSA based on the whether SSA was utilized for the prior release (see below).
When enabled, Helm will submit object to the Kubernetes API server via SSA patches.

If SSA is utilized for a release, this will be stored as a field in the release's metadata.
If the field is unset when reading a prior release, this implies that the release was created with an old version of Helm utilizing CSA.

```
type Release struct {
   ApplyMethod *string  `json:"apply_method,omitempty"` // "ssa" OR "csa"
}
```

If a user deploys a chart with a new version of Helm with SSA enabled, then upgrades the chart with an older version of Helm. It is expected that the older Helm will be able to upgrade the chart with CSA. And the new release's metadata will omit the indication of SSA usage (and so further upgrades/rollbacks will continue to use CSA by default).

In a future version of Helm 4 CLI, it might be desirable to omit a warning if a user is not using SSA (in order to prompt the user to enabling SSA for further releases).

#### Custom resources

Helm will utilize SSA for custom resources, which it previously [didn't merge](https://github.com/helm/helm/blob/1b260d0a796882a1e90f0fa3c832659dbe2e675c/pkg/kube/converter.go#L54-L69).
As SSA allows custom resources to be correctly merged based on metadata within the CRD.
Or the API server will use the default approach (<https://kubernetes.io/docs/reference/using-api/server-side-apply/#custom-resources>).

#### Conflicts and forcing

It is possible that when Helm upgrades (or rollbacks) a chart, there will be a field conflict with another field manager.
In this case, Helm will report the error for the conflict field(s) to the user.

Helm will also add a new flag `--force-conflicts=false|true` to the install, upgrade and rollback commands (modeled after the life `kubectl apply` flag)
And corresponding API objects field.
When `true` (default `false`), Helm will enabling override the field conflicts, setting field(s) to the charts values.
And also make Helm the field manager for the conflicting field(s).
The `--force-conflicts` flag documentation should explicity mention the downsides of conflict forcing ie. the chart update is overriding object values that another process is expecting to own/manage.

For more details, see the _Overwrite value, become sole manager_ of <https://kubernetes.io/docs/reference/using-api/server-side-apply/#conflicts>

(Of note, Helm upgrade already includes the `--force` flag, which tells Helm to replace objects during upgrade.
However, given this different semantics of `--force`, a new flag is required)

An additional special case is rollbacks on failed upgrade (`--atomic` flag). It is possible for Helm to attempt to rollback to a version which now has a conflict.
For which the rollback will fail.
Since rollbacks are "best effort" and can fail for other reasons as well, it seems best to treat this new rollback failure-case similarly (report to the user via logging and error).
If the user specified `--force-conflicts` to the upgrade, the corresponding rollback should similarly force conflicts (this is the behavior of `--force`).
If the rollback is to a release version which previously didn't have SSA enabled, the rollback install should also disable SSA.

#### Special

- `--dry-run=client` won't work with SSA enabled
- If the user attempts to use SSA with a cluster which does not support it (unlikely: SSA [went GA](https://kubernetes.io/blog/2021/08/06/server-side-apply-ga/) in Kubernetes v1.22), Helm should error
- Client-side API object validation isn't applicable when SSA enabled (`helm install|upgrade --disable-openapi-validation`)
- Hooks could be deployed with SSA. But as hook objects are not updated (create/delete only), they are not really affected
- Stored release manifests should not change (they store the object as Helm intends)

### Implementation

Helm will send object's manifests to the Kubernetes API server via SSA patches. Similar to `kubectl apply`.
Utilizing the field manager name `"helm"`:

https://github.com/kubernetes/kubectl/blob/197123726db24c61aa0f78d1f0ba6e91a2ec2f35/pkg/cmd/apply/apply.go#L248
https://github.com/kubernetes/kubectl/blob/197123726db24c61aa0f78d1f0ba6e91a2ec2f35/pkg/cmd/apply/apply.go#L564-L602

## Backwards compatibility

<!-- Describe potential impact and severity on pre-existing code. -->

SSA should be generally compatible functionality-wide with CSA.
The expectation is that SSA should generally produce equivalent Kubernetes objects as CSA.

There four main behavioural differences for Helm switching to SSA:

1. The differences between object's manifests managed with SSA vs the existing three-way strategic merge process:
   - Arrays will be merged
   - Removal of unset fields
   - Default values
2. Issues with a user enabling/disabling SSA over the lifetime of a release e.g. environment differences or older Helm clients which don't understand SSA
3. SSA will apply to custom resources
4. Field management ownership conflicts

For 1., chart-operators may opt-out out of SSA.
For 2., the user can control by ensuring SSA is consistently used within their environment.
For 3., the it is generally considered a limitation of Helm not updating custom resources, for which SSA is expected to be an improvement.
For 4., the `--force-conflicts` flag exists.

Additionally as noted, SSA isn't compatible with very old Kubernetes versions (Kubernetes v1.17 and prior)

## Security implications

<!-- How could a malicious user take advantage of this new feature? -->

There should not be any additional security concerns.
Server-side apply and client-side apply should be equivalent security-wise.

## How to teach this

<!-- How to teach users, new and experienced, how to apply the HIP to their work. -->

Documentation for how Helm does object upgrades, and the advantages of SSA.
Also documentation for the `--server-side=false|true|auto` and `--force-conflicts=false|true` flags, and the defaults for install/upgrades/rollbacks.
And the corresponding documentation for the field(s) SDK API objects.

## Reference implementation

<!-- Link to any existing implementation and details about its state, e.g.
proof-of-concept. -->

Kubectl:
- <https://github.com/kubernetes/kubectl/blob/197123726db24c61aa0f78d1f0ba6e91a2ec2f35/pkg/cmd/apply/apply.go#L248>
- <https://github.com/kubernetes/kubectl/blob/197123726db24c61aa0f78d1f0ba6e91a2ec2f35/pkg/cmd/apply/apply.go#L564-L657>

Prototype (hack) for Helm:
- <https://github.com/helm/helm/compare/main...gjenkins8:helm:server_side_apply_proto>


## Rejected ideas

<!-- Why certain ideas that were brought while discussing this HIP were not
ultimately pursued. -->

### Chart field manager support

This HIP defers including field manager support in charts.
Helm could enabling apply objects with a distinct field manager names.
e.g. allowing a chart to supply multiple yaml manifest documents for the same object.

It is unclear how Helm users will interact with such a feature e.g. the chart developer might want/need to specify the objects field manage name in a chart.
And `helm template` might somehow need to allow describing the object's field manager name in its output.

Once more information is gathered on how chart developers might utilze explicit field manager support, a proposal introducing field manage support for charts could be written.

### Field ownership transfer

Helm won't support ownership transfer of fields.
Similar to the `kubectl` example: <https://kubernetes.io/docs/reference/using-api/server-side-apply/#transferring-ownership>.

Dissimilar to `kubectl`, is Helm's distinguishes between chart developers and chart operators.
Chart developers may not consider, or may not even know, which fields may be overwritten (managed by) by another process in a user's Kubernetes cluster.

Ownership transfer would likley first require charts have field manager support.
As well as additional support for per-object conflicts resolution.

## Open issues

<!-- Any points that are still being decided/discussed. -->

## References

<!-- A collection of URLs or materials used as references through the HIP. -->

- <https://kubernetes.io/docs/reference/using-api/server-side-apply/>
- <https://github.com/kubernetes/enhancements/tree/master/keps/sig-cli/3805-ssa-default#summary>
- <https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md>
- <https://github.com/kubernetes-sigs/structured-merge-diff>
- <https://pkg.go.dev/k8s.io/apimachinery/pkg/util/strategicpatch#CreateThreeWayMergePatch>
- <https://kubernetes.io/docs/reference/using-api/server-side-apply/#conflicts>