---
hip: 9999
title: "container tools specifications for OCI registry management"
authors: [ "George Jenkins <gvjenkins@gmail.com>", "Andrew Block <andy.block@gmail.com>" ]
created: "2025-02-16"
type: "feature"
status: "draft"
helm-version: "4"
---

## Abstract

The [container tools][containers-tools-project] project defines an alternative specification for managing client OCI registry configuration that supports more advanced features. Compared to Docker's `docker/config.json` that Helm currently uses today.

Pertainently these include [registries.conf][registries-conf], [auth.json][auth-json], as well as the other specifications in <https://github.com/containers/container-libs/blob/main/image/docs/>.

This HIP focuses on the initial implementation using ORAS to supports containers' OCI registry mamagement to supersede `docker/config.json` within Helm.
Today, `auth.json` provides the equivalent functionality to `docker/config.json` for storing OCI registry credentials.
Further HIPs will be created to expose additional functionality based on utilizing `registries.conf` and other container tools configurations.

[containers-tools-project]: https://github.com/containers "container tools project"
[registries-conf]: <https://github.com/containers/container-libs/blob/main/image/docs/containers-registries.conf.5.md> "registries.conf specification"
[auth-json]:       <https://github.com/containers/container-libs/blob/main/image/docs/containers-auth.json.5.md> "auth.json specification"

## Motivation

Helm currently uses Docker's `docker/config.json` to store client OCI registry configuration today.
Limited to mapping a registry domain to authentication credentials only.

The [CNCF-hosted](https://www.cncf.io/projects/podman-container-tools/) has created several specification for managing client OCI registry configuration.
These are `registries.conf`, `auth.json`, `policy.json`, etc. Which provide much more advanced functionality for client OCI registry management than `docker/config.json`.

Notably:

- support for repository prefixes (allowing different credentials for different prefixes)
- registry mirrors
- registry "aliasing"—features
- allowing/denying registries
- signing and verification policies

Helm would like to introduce features that utilize these functionalities.
Supporting containers' configuration specifications would enable these features without requiring Helm to create or implement its own mechanisms.

## Rationale

Utilizing an existing specification and libraries enables Helm to immediately build upon an existing standard, rather than inventing its own convention.

The container tools project specifications were picked as being a format intended for consumption beyond the Docker application container ecosystem.

ORAS (the library Helm uses for supporting OCI functionality) has planned support for `registries.conf`, `auth.json`, etc for client OCI registry management.

Helm will fall back or prefer to `docker/config.json` for registry authentication (see below for options) to ensure existing user workflows remain functional.

## Specification

When enabled, Helm will utilize `auth.json` for registry credentials and `registries.conf` for registry feature management:
<https://github.com/containers/container-libs/blob/main/image/docs/containers-auth.json.5.md>
<https://github.com/containers/container-libs/blob/main/image/docs/containers-registry.conf.json.5.md>

Helm will write to `auth.json` when performing OCI registry operations that modify client OCI registry configuration (e.g., `helm registry login`). And as long as the operation can be supported by `docker/config.json`, Helm will dual-write to `docker/config.json` for compatibility purposes. Dual-write will be removed in a future version of Helm, and Helm will write only to `auth.json`.

Helm will use the same filepath ordering when searching for matching OCI entries in `registries.conf` and credentials in `auth.json` as described in the respective specifications.

`auth.json`:

1. Location specified in environment variable `REGISTRY_AUTH_FILE` if set
2. `${XDG_RUNTIME_DIR}/containers/auth.json` on Linux; `$HOME/.config/containers/auth.json` on Windows and macOS

`registries.conf`:

1. `$HOME/.config/containers/registries.conf` if it exists
2. Otherwise `/etc/containers/registries.conf`

Helm will use ORAS v3 for updating (and reading) `registries.conf` / `auth.json`, working with the ORAS project to build support.
Container tools OCI config support won't move out of experimental until support is deemed sufficent (see below).
Helm must utilize any `credHelpers` specified in `auth.json`.

If reading a registry configuration from `registries.conf` or `auth.json` results in a configuration that Helm does not support, Helm must ignore that entry (e.g., a `location` field or non-empty URI path).
As ORAS gains support for additional `registries.conf`, `auth.json`, etc features, Helm will also gain support for these features.
Helm must ensure these features do not break backwards compatibility guarantees.

If an error occurs while reading `registries.conf` or `auth.json`, etc, Helm must report an error to the user.
Otherwise, users who expect configuration from `registries.conf` / `auth.json`, etc to be effective may encounter unexpected fallback behavior.

Helm must expect (and even encourage) users to utilize other tooling to manage `registries.conf`, `auth.json`, etc.

## `registries.conf`/`auth.json` vs. `docker/config.json` preference

To manage the release, Helm will introduce an environment variable `HELM_EXPERIMENTAL_OCI_CONTAINERS_CONFIG`.
Initially, when unset or set to `false`, Helm will continue to use only `docker/config.json`.
When set to `true`, Helm will enable `registries.conf` / `auth.json`, etc support as described herein.

Once stable, Helm will default the unset behavior of `HELM_EXPERIMENTAL_OCI_CONTAINERS_CONFIG` to enable `registries.conf` / `auth.json`, etc support.
And eventually remove the environment variable altogether. Potentially requiring Helm v5 (next major version of Helm) to do both.

### Example: basic login

For example, a registry login command:

```bash
helm registry login "registry.example.com" --username foo --password bar
```

Will result in the `auth.json` excerpt:

```json
# auth.json
"auths": {
  ...
  "registry.example.com/theprefix": {
    "auth": "Zm9vOmJhcgo="
  }
}
```

## Backwards compatibility

Helm's fallback to Docker's registry configuration ensures the vast majority of existing user workflows remain the same.

However, there are two potential incompatibility scenarios:

1. A corrupt `registries.conf` / `auth.json`, etc will cause an error for existing workflows
2. An invalid or incompatible with Helm `registries.conf` / `auth.json` entry for the given OCI registry will cause the user's workflow to fail

The first two can be mitigated by users ensuring their system's `registries.conf` / `auth.json` are valid, and only including configuration options Helm supports for the registries they plan to use with Helm.

## Security implications

- Support for signing and access control policy `policy.json` should improve OCI access management
- `auth.json` introduces a new mechanism for storing OCI credentials, which may introduce credential management vulnerabilities specific to `auth.json`
- `registries.conf` can introduce mirroring and redirection of OCI requests

## How to teach this

Helm's documentation will need to be updated with details of Helm's `registries.conf` / `auth.json` support and fallback to `docker/config.json`.

## Reference implementation

- TODO: link to ORAS v3 `registries.conf` / `auth.json` implementation, in progress
- TODO: create experimental PR/branch for Helm implementing this HIP

## Rejected ideas

### Falling back to Docker’s config upon error reading registries.conf

Falling back upon error means users who expect their configuration to be taken from `registries.conf` will unexpectedly have OCI configuration taken from Docker, potentially resulting in difficult to diagnose failures authenticating to the repository.

Rather than "failing fast" and requiring users to ensure their configuration is valid.

## Open issues

- ORAS `registries.conf` support: <https://github.com/oras-project/oras-go/issues/918>
- Is the release and backwards compatibility plan good enough?
- ~~Discuss Helm's/ORAS's potential usage of `registries.conf` with `registries.conf` owners~~

## References

### ORAS credential store (Docker registry configuration) docs

<https://pkg.go.dev/oras.land/oras-go/v2/registry/remote/credentials#NewStoreFromDocker>

### `registries.conf` / `auth.json` specifications

<https://github.com/containers/container-libs/blob/main/image/docs/containers-registries.conf.5.md>
<https://github.com/containers/container-libs/blob/main/image/docs/containers-auth.json.5.md>
