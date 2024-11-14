---
hip: 9999
title: "Utilize Server Side Apply for installs/upgrades"
authors: [ "George Jenkins <gvjenkins@gmail.com>" ]
created: "2023-07-07"
type: "feature"
status: "draft"
---

## Abstract

<!-- A short (~200 word) description of the technical issue being addressed. -->
This HIP proposes for Helm to support Kubernetes' [Server-Side Apply](https://kubernetes.io/docs/reference/using-api/server-side-apply/) (SSA) for [object](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-object) management.

Helm 3 introduced the [three-way strategic merge and patch](https://helm.sh/docs/faq/changes_since_helm2/#improved-upgrade-strategy-3-way-strategic-merge-patches). Allowing Helm to update objects which have been modified by other processes. Kubernetes now offers a similar server-side process that has several advantages over the client-side apply (CSA) methods that Helm and kubectl (for example) have traditionally utilized.

The scope of this HIP is limited to incorporating SSA as an opt-in feature only. Deferring changing user's existing usage of the (client-side) three-way strategic merge patch processes (without opt-in). And considers how to deal with field conflicts and other compatibility concerns that may arise with the differences between SSA and CSA.

## Motivation

<!-- Clearly explain why the existing design is inadequate to address the problem
that the HIP solves. -->

The primary motivation for Helm to switch to Server-Side Apply is to gain the benefits of Kubernetes managing object upgrades. Where objects may have been modified by another process. The following describes the advantages of SSA from Kubernetes perspective, where SSA generally is considered to be superior:

https://kubernetes.io/blog/2022/10/20/advanced-server-side-apply/

Helm supporting SSA will allow users to realize the benefits of SSA. And longer term goals (out of scope for this HIP) might be for Helm to utilize SSA by default ([similar to kubectl](https://github.com/kubernetes/enhancements/tree/master/keps/sig-cli/3805-ssa-default)). Finally later as a code maintenance benefit, eventually drop support for the strategic-merge patch client-side apply process/code.

## Rationale

<!-- Describe why particular design decisions were made. -->

Due to the backwards compatibility concerns documented below, the decision was made to provide opt-in support only. And even when a chart is deployed with SSA enabled, Helm should maintain full compatibility with reverting back to using CSA for upgrades/rollbacks. Additionally it was desirable to mimic kubectl's cli interface/flags. As a way for Helm to be consistent with an existing kubectl functionality.

## Specification

<!-- Describe the syntax and semantics of any new feature. -->

### General

Helm will add a new flag to the install, upgrade and rollback sub-commands (modelled after the like `kubectl` flag). And add a corresponding field to the respective install/upgrade/rollback SDK API objects:

`--server-side=false|true|auto`

Helm will default to `false` for installs, retaining its existing CSA behavior.  And `auto` for upgrade and rollbacks, where Helm will enable SSA based on the whether SSA was utilized for the prior release. With the usage of SSA for a given release being stored in the release's metadata. When enabled, Helm will submit object to the Kubernetes API server via SSA patches.

If a user deploys a chart with a new version of Helm with SSA enabled, then upgrades the chart with an older version of Helm. It is expected that the older Helm will be able to upgrade the chart with CSA. And the new release's metadata will omit the indication of SSA usage (and so further upgrades/rollbacks will continue to use CSA by default).

In a future version of Helm 3 cli, it might be desirable to omit a warning if a user is not using SSA (in order to prompt the user to enabling SSA).

#### Custom resources

Helm will utilize SSA for custom resources, which it previously [didn't merge](https://github.com/helm/helm/blob/1b260d0a796882a1e90f0fa3c832659dbe2e675c/pkg/kube/converter.go#L54-L69). As SSA allows custom resources to be correctly merged based on metadata within the CRD. Or the API server will use the default approach (<https://kubernetes.io/docs/reference/using-api/server-side-apply/#custom-resources>).

#### Conflicts and forcing

It is possible that when Helm upgrades a chart, there will be a field conflict with another field manager. In this case, Helm should report the error to the user.

Helm will also add a new flag `--force-conflicts` to the `upgrade` and `rollback` commands (and corresponding API object field). Which will tell the apply to ignore/override field conflicts (and so thus make Helm the field manager).

(Of note, Helm upgrade already includes the `--force` flag, which tells Helm to replace objects during upgrade. However, given this different semantics of `--force`, it is thought a new flag is required)

An additional special case is rollbacks on failed upgrade (`--atomic` flag). It is possible for Helm to attempt to rollback to a version which now has a conflict. For which the rollback will fail. Since rollbacks are "best effort" and can fail for other reasons as well, it seems best to treat this new rollback failure-case similarly (report to the user via logging and error). If the user specified `--force-conflicts` to the upgrade, the corresponding rollback should similarly force conflicts (this is the behavior of `--force`). If the rollback is to a release version which previously didn't have SSA enabled, the rollback install should also disable SSA.

##### [Optional] Transferring ownership

_this section is optional, and/or for HIP discussion_

Helm could also allow acceptance of conflicts and ownership transfer of fields. Similar to the `kubectl` example: https://kubernetes.io/docs/reference/using-api/server-side-apply/#transferring-ownership

For Helm, options include:

0. Do nothing: (initially) Given the unknowns of how conflicts will affect users in practice, and the potential complexity of implementing one of the below options. Defer implementing anything until feedback from users can be gathered
1. Chart objects could be annotated with a special annotation: e.g. `helm.sh/accept-conflicts: true`. Helm would apply objects with a distinct field manager name (allowing multiple yaml documents for the same object), and ignore conflicts for the given apply
2. A cli flag (and corresponding API field): `--accept-conflicts-from=<field managers...>`. In this case, Helm would simply accept conflicts from the respectively named field manager (TODO: does client-go / Kubernetes API even easily allow this; how would the users intended changes actually get applied)
3. Allow the releases operator to specify "patches" to a chart

Dissimilar to `kubectl`, is Helm's distinguishes between Chart developers and Chart operators. Chart developers may not consider, or may not even know, which fields may be overwritten (managed by) by another process in a user's Kubernetes cluster.

As such, while option 1. is probably the cleanest from an implementation and user understanding perspective. It may have limitations from a operational perspective. Option 2. may not even be technically possible (it is included as a thought on how to give the Chart's operator control). And 3. introduces significant complexity for the operator of the chart (and perhaps Helm).

#### Special

- `--dry-run=client` won't work with SSA enabled
- If the user attempts to use SSA with a cluster which does not support it (unlikely: SSA [went GA](https://kubernetes.io/blog/2021/08/06/server-side-apply-ga/) in Kubernetes v1.22), Helm should error.
- Client-side API object validation isn't applicable when SSA enabled (`helm install|upgrade --disable-openapi-validation`)
- Hooks could be deployed with SSA. But as hook objects are not updated (create/delete only), they are not really affected
- Stored release manifests should not change (they store the object as Helm intends)

### Implementation

Helm will send object's manifests to the Kubernetes API server via SSA patches. Similar to `kubectl`. Utilizing the field manager name `"helm"`:

https://github.com/kubernetes/kubectl/blob/197123726db24c61aa0f78d1f0ba6e91a2ec2f35/pkg/cmd/apply/apply.go#L248
https://github.com/kubernetes/kubectl/blob/197123726db24c61aa0f78d1f0ba6e91a2ec2f35/pkg/cmd/apply/apply.go#L564-L602

## Backwards compatibility

<!-- Describe potential impact and severity on pre-existing code. -->

There are thought to be four main compatibility concerns for Helm switching to SSA:
1. The differences between object's manifests managed with SSA vs the existing three-way strategic merge process:
   - Arrays will be merged
   - Removal of unset fields
   - Default values
2. Issues with a user enabling/disabling SSA over the lifetime of a release e.g. environment differences or older Helm clients which don't understand SSA
3. SSA will apply to custom resources

For 1., this HIP limits Helm to opt-in to server side apply, allowing users to verify SSA's behavior matches their expectations. For 2., the user can mitigate by ensuring SSA is consistently used within their environment. But the expectation is that SSA should generally produce equivalent Kubernetes objects as CSA.

Additionally as noted, SSA isn't compatible with very old Kubernetes versions (Kubernetes v1.17 and prior)

## Security implications

<!-- How could a malicious user take advantage of this new feature? -->

There should not be any additional security concerns. Server-side apply and client-side apply should be equivalent security-wise.

## How to teach this

<!-- How to teach users, new and experienced, how to apply the HIP to their work. -->

Documentation for how Helm does object upgrades, and the advantages of SSA. Also documentation for the `--server-side=false|true|auto` and `--force-conflicts=false|true` flags, and the defaults for install/upgrades/rollbacks. And the corresponding documentation for the field(s) SDK API objects.

## Reference implementation

<!-- Link to any existing implementation and details about its state, e.g.
proof-of-concept. -->

Kubectl: \
<https://github.com/kubernetes/kubectl/blob/197123726db24c61aa0f78d1f0ba6e91a2ec2f35/pkg/cmd/apply/apply.go#L248>
<https://github.com/kubernetes/kubectl/blob/197123726db24c61aa0f78d1f0ba6e91a2ec2f35/pkg/cmd/apply/apply.go#L564-L657>

Prototype (hack) for Helm: \
<https://github.com/helm/helm/compare/main...gjenkins8:helm:server_side_apply_proto>


## Rejected ideas

<!-- Why certain ideas that were brought while discussing this HIP were not
ultimately pursued. -->

- Opting users into SSA as default (defer to later)
- Requiring upgrades to be follow install time CSA or SSA setting: users may an old Helm versions. Or may want to "upgrade" an existing release with SSA.

## Open issues

<!-- Any points that are still being decided/discussed. -->

- How to best allow Helm/users to deal with conflicts (Transferring Ownership)

## References

<!-- A collection of URLs or materials used as references through the HIP. -->

- <https://kubernetes.io/docs/reference/using-api/server-side-apply/>
- <https://github.com/kubernetes/enhancements/tree/master/keps/sig-cli/3805-ssa-default#summary>
- <https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md>
- <https://github.com/kubernetes-sigs/structured-merge-diff>
- <https://pkg.go.dev/k8s.io/apimachinery/pkg/util/strategicpatch#CreateThreeWayMergePatch>