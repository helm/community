# Security Considerations

This architecture for Helm is more secure than Helm 2 (and equally secure to
Helm Classic). The following security goals are met by this proposal:

- Helm now runs with the ID and permissions of the UserAccount or
  ServiceAccount executing the commands.
- RBAC can now be applied to each user
- On-the-wire security is as secure as the cluster is configured to be (e.g. if
  KUBECONFIG is set up for mTLS, that's what we use)
- Auditability is now a feature of the cluster, as each chart is "created by"
  the user who runs the request
- In-cluster CRDs can be secured with RBAC, too, providing access controls for
  releases.
