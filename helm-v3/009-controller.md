# Appendix A: A Helm Controller

> This section is a placeholder until a detailed proposal is written.

This section describes a second implementation of Helm. The goal of this
implementation is to provide a controller/CRD façade for the core Helm
features. _However, defining the specifics of this controller was deemed
outside of the scope of this proposal, which focuses on the reference
implementation of the Helm CLI_.

A client-only Helm follows a push-based model. To facilitate a pull-based
model, we can also create a Helm controller that provides a CRD facade, but
uses the Helm library.

It is suggested that this (a) be a core Helm project, but (b) is kept
separately from the main Helm codebase.

This controller accepts Chart and values configuration as a new `HelmRequest`
CRD, and then provides the following operations:

- Install
- Upgrade
- Delete
- Get
- List

Just like the client, the controller will operate on Release and release
version Secret. For that reason, the Helm CLI and the Helm CRD can be used in
conjunction, and RBACs can even be applied to lock down certain features for
one or the other. (For example, Helm CLI may be restricted to only read
functions, while the controller can perform read and write operations)

> [name=Adnan Abdulhussein] How will the controller apply a user's RBAC
> permissions when installing a chart? Or is this not possible, and the
> controller will need to be restricted using service accounts?  [name=Matt
> Fisher] It is not possible to apply a user's RBAC permissions when installing
> a chart via this model, which is why we decided to remove Tiller in Helm 3.

This implementation will necessarily have some limitations and restrictions
that the client Helm does not; namely, it will not be able to handle large
charts because there is a 1M limit to CRD size.

It is also assumed that the controller model may not choose to implement a
number of the client features, such as dry runs, repository management, chart
creation, etc.
