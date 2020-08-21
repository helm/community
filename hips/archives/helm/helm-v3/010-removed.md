# Appendx C: Removed Sections

The following parts of the proposal have been removed.

## Overlays

An earlier draft included a provision for overlaying templates within charts. This section
was removed once we realized that the same thing can be easily accomplished using
the `post-render` event and some trivial Lua.

This appendix will be removed before the final proposal.

## Chart.yaml refactored as Kubernetes resource

An earlier draft proposed formatting Chart.yaml as a Kubernetes resource. However,
it became evident that there was no intention of installing the Chart.yaml into
the cluster, at which point it did not make sense to retain that functionality.

## Application CRD

Earlier drafts contained information about implementing the Application CRD
proposal as a feature of Helm.

While there is a plan to support Application CRD in Helm, it will not be a
required feature of Helm. For that reason, it has been removed from the proposal.

## Alternatives to Lua extensions

The following alternatives were proposed:

- Implement lifecycle event handlers as containers in-cluster
- Reimplement Helm in Python|Ruby|TypeScript|JavaScript
- Extend Plugins to be event handlers
