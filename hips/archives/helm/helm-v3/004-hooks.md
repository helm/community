# Hook Annotations

In Helm 2, hook annotations could be placed on various manifests, and those
annotations would result in special treatment.

In Helm 2, resources with hook annotations were unmanaged. In Helm 3, the owner
reference is set on those, and they will be removed when the chart is deleted.
There will be a way to override the management policy by setting an annotation
for `helm.sh/hook-policy: unmanaged`

> [name=Taylor Thomas] Would hooks be needed anymore with the new eventing
> system? I think all we would need is something that can translate Helm v2
> hooks into the new events behind the scenes
