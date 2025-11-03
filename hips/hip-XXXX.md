---
hip: 9999
title: "registries.conf support for OCI registry management"
authors: [ "George Jenkins <gvjenkins@gmail.com>", "Andy Block <andy@redhat.com>" ]
created: "2025-02-16"
type: "feature"
status: "draft"
---

## Abstract

[registries.conf][registries-conf] is an alternative specification for managing client OCI registry configuration that supports more advanced features, compared to Docker's `docker/config.json` that Helm currently uses today.

This HIP focuses on the initial implementation of supporting `registries.conf` within Helm.
Further HIPs will be created to expose additional functionality based on utilizing `registries.conf`.

[registries-conf]: <https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md> "registries.conf specification"

## Motivation

Helm uses Docker's `docker/config.json` to store client OCI registry configuration today.

`registries.conf` provides much more advanced functionality for client OCI registry management than `docker/config.json`.
Notably:

- support for repository prefixes (allowing different credentials for different prefixes)
- registry mirrors
- registry "aliasing"—features

Helm would like to introduce features depending on these functionalities. Supporting `registries.conf` would enable them without Helm having to create or implement its own mechanisms.
With `registries.conf` having been standardized by other container ecosystem CLI tools (podman, buildah, skopeo, etc.).

## Rationale

Utilizing an existing specification and libraries enables Helm to immediately build upon an existing standard, rather than inventing its own convention.

`registries.conf` was picked as being a format intended for consumption beyond the Docker application container ecosystem.

ORAS (the library Helm uses for supporting OCI functionality) has planned support for `registries.conf` client-side OCI registry management.

## Specification

Helm will utilize the `registries.conf` specification when determining OCI registry information (authentication credentials, etc.):
<https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md>

`registries.conf` will be preferred either when a `registries.conf` file already exists on the user's system, or when Helm supports and a user utilizes functionality that cannot be supported by Docker's configuration file in the future.

<!-- The usage of `registries.conf` will be forced, when in the future -->

For example, a registry login command:

```bash
helm registry login "oci.example.com" --username foo --password bar
```

Will result in the configuration excerpt (if, and only if, `registries.conf` exists on the user's local system):

```toml
# registries.conf
[[registry]]
prefix = "oci.example.com"
```

```json
# auth.json
"auths": {
  ...
  "oci.example.com": {
    "auth": "Zm9vOmJhcgo="
  }
}
```

Helm will use ORAS v3 for updating (and reading) `registries.conf` (TODO: link to ORAS v3 `registries.conf` implementation).

If reading a registry configuration from `registries.conf` results in a known configuration that Helm doesn't support, Helm must report a warning (e.g., `location` or non-empty URI path in `prefix`).

An error reading `registries.conf` must result in an error for the user. Otherwise, users who expect configuration from `registries.conf` to be effective will have an unexpected fallback.

To account for existing configuration in Docker’s registry configuration, Helm will prefer `registries.conf` when resolving OCI registries (including credentials), and fall back to the existing storage mechanism if `registries.conf` does not contain an entry for the required OCI registry (including if `registries.conf` does not exist).

Helm must expect (and even encourage) users to utilize other tooling to manage `registries.conf`.

## Backwards compatibility

Helm's fallback to Docker's registry configuration ensures the vast majority of existing user workflows remain the same.

However, there are three potential incompatibility scenarios:

1. A corrupt `registries.conf` will cause an error for existing workflows
2. An invalid or incompatible with Helm `registries.conf` entry for the given OCI registry will cause the user's workflow to fail
3. Helm’s preference for `registries.conf` will break users who assume credentials are stored in Docker’s registry configuration

The first two can be mitigated by users ensuring their system's `registries.conf` is valid, and only includes configuration options Helm supports for the registries they plan to use with Helm.

The last is mitigated by not using `registries.conf` initially unless it exists on the user's system.

## Security implications

`registries.conf` introduces a new mechanism for storing OCI credentials, which may introduce credential management vulnerabilities specific to `registries.conf` / `auth.json`

Transitive dependencies of the TBD package for managing `registries.conf` may introduce security scanner noise (which tends to be a problem in the container library ecosystem)

## How to teach this

Helm's documentation will need to be updated with details of Helm's `registries.conf` support and fallback to Docker config.

## Reference implementation

<!--
Link to any existing implementation and details about its state, e.g.
proof-of-concept.
-->

## Rejected ideas

### Falling back to Docker’s config upon error reading registries.conf

Falling back upon error means users who expect their configuration to be taken from `registries.conf` will unexpectedly have OCI configuration taken from Docker, potentially resulting in difficult to diagnose failures authenticating to the repository.

Rather than "failing fast" and requiring users to ensure their configuration is valid.

## Open issues

- ORAS `registries.conf` support: <https://github.com/oras-project/oras-go/issues/918>
- Is the release and backwards compatibility plan good enough?
- <strike>Discuss Helm's/ORAS's potential usage of `registries.conf` with  `registries.conf` owners</strike>

## References

### ORAS credential store (Docker registry configuration) docs

<https://pkg.go.dev/oras.land/oras-go/v2/registry/remote/credentials#NewStoreFromDocker>

### registries.conf specification

<https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md>
