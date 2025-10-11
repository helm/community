---
hip: 9999
title: "Forward compatibility: Chart.yaml minimumHelmVersion"
authors: [ "George Jenkins <gvjenkins@gmail.com>" ]
created: "2024-11-18"
type: "feature"
status: "draft"
helm-version: "4"
---

## Abstract

This HIP proposes supporting a `minimumHelmVersion` field in `Chart.yaml`.
By enabling this feature, an error can be raised if the Helm version used to operate on the chart is below the declared version.
This allows Helm to provide forward compatibility guarantees for Helm features/functionality over time.


## Motivation

Helm has no mechanism for a chart to declare the minimal version of Helm required for the chart to install/update correctly.
As such, it is invalid to release a chart that utilizes features/fixes included in newer versions of Helm.

At best, any incompatibility will be detected and there will be an explicit failure (and the user will be notified with an error).
But potentially an incompatibility may go undetected, and no hard error will be presented.

While the hard-failure case is better, it still requires the user to debug and realize the failure is due to a version mismatch.
The second "soft-failure" case is much worse, as it could lead to unexpected behavior in the deployed chart application, likely requiring a much more involved debugging requirement and confusing user experience.

Providing a mechanism for a chart to prescribe the minimum versions of Helm for the chart's feature set will enable chart developers to prevent indirect errors for chart consumers/operators.

Supporting a simple SemVer based `minimumHelmVersion` field in chart's `Chart.yaml` is thought to be a simple way to provide forward compatibility guarantees.


## Rationale

It is thought that over time, as Helm extends its functionality, there will be greater scope for forward compatibility errors to present themselves.

And in particular, the mechanism that describes the minimum feature set must be built into and understood by prior versions of Helm.

Utilizing a simple minimum version SemVer field in a chart is thought to be a simple and succinct way to enable forward compatibility guarantees for future versions of Helm.

Similar to the way the Go toolchains allow forward compatibility guarentees via [`go.mod`][1].


## Specification

`Chart.yaml` will include an optional field `minimumHelmVersion`, which is a [SemVer][2] string for the minimum version of Helm required for the chart to operate:

```
minimumHelmVersion: <MAJOR>[.<MINOR>[.<PATCH>]]
```

The minor and patch fields are optional.
And will be inferred as zeros if not specified.

When loading a chart, if the `minimumHelmVersion` exists, Helm will verify its version exceeds or equals the version constraint specified in the field.

When "strict" loading of `Chart.yaml` is used: ie. the loading of `Chart.yaml` errors when unknown fields are present, Helm will first 'peek' into `Chart.yaml` and extract the `minimumHelmVersion` field (if it exists) and perform the version constraint check at this point.
This is to provide better UX than simply raising an error on any potential new fields in `Chart.yaml` that are unknown to the current version of Helm.

## Backwards compatibility

As `minimumHelmVersion` is an optional field, it will be inferred to be unset in existing charts.
And when unset, Helm will retain existing behavior (no check will be implemented)

Older versions of Helm which do not understand the `minimumHelmVersion` field will ignore the field even if set (as they load `Chart.yaml` ignoring unknown fields).
However, there isn't anything that can practically be done to address this encoded behavior.


## Security implications

None


## How to teach this

Update docs for `Chart.yaml`


## Reference implementation

N/A


## Rejected ideas

**Generic version contraints:**
It is thought a "minimum" constraint is preferred over a generic "constraint" system.
ie. where a user could list multiple version constraints to be satisfied
Similar to e.g. [pip's requirement-specifiers][3].
This is because Helm makes great effort to remain [backwards compatibility][4].
And so it is presumed unnecessary that a user would need to specify anything but a single minimum version constraint.
And in the future when Helm does make breaking changes to charts, it is expected the chart API version will be incremented to explicitly signify this new incompatiblity.

**Capabilities:**
Providing a capabilities based system.
Rather than just directly inferring Helm capabilities from its version.

A `Chart.yaml` could describe its required features ie. capabilities it expects/requires the Helm version to provide.
Helm could inspect and match against features/capabilities that the specific version does provide.
However, since core Helm capabilities are only every additive due to Helm's [backwards-compatibility rules][4], a capabilities based system is redundant.

*NB: at the time of writing, plugins for extending Helm's functionality are being heavily discussed.
And while plugins will likely allow extending capabilities independent of the Helm version, it is presumed plugins will include their own version constraint specifier system.
That will ensure conditions for valid plugin versions are met*

### Future ideas

Extending `helm lint` to detect incompatible chart features for the currently specified `minimumHelmVersion`.


## Open issues

N/A


## References

[1]: <https://go.dev/blog/toolchain#forward> "Golang forward compatibility"
[2]: <https://semver.org> "Semantic versioning"
[3]: <https://pip.pypa.io/en/stable/reference/requirement-specifiers/#requirement-specifiers> "Pip request-specifiers"
[4]: <hip-0004.md> "hip-0004: Document backwards-compatibility rules"
