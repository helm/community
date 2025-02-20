---
hip: 9999
title: "registries.conf support for OCI registry management"
authors: [ "George Jenkins <gvjenkins@gmail.com>", "Andy Block <andy@redhat.com>" ]
created: "2025-02-16"
type: "feature"
status: "draft"
---

## Abstract

[registries.conf][registries-conf] is a an alternative specification (to Docker's `docker/config.json`) for managing OCI registry configuration. That supports more advanced features, and has been standardized by other container ecosystem CLI tools (podman, buildah, skopeo, etc)

Support would extend Helm's flexilbity with respect to OCI configuration, including repository prefixes and aliases. This HIP focuces on the implementation of supporting `registries.conf`. Further HIPs to be created for exposing functionality based on utilzing `registries.conf`.

[registries-conf]: <https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md> "registries.conf specification"

## Motivation

`registries.conf` provides much more flexibible support for OCI registry management than the current Docker CLI based configuration format that Helm uses today.

Including support for repository prefixes (allowing different credentials for different prefixes), registry mirrors, and registry "aliasing". Features Helm would like to introduce. But is currenly blocked by a lack of mechanism to store detail

(in particular, the existing registry configuration Helm uses, Docker’s `$HOME/.docker/config.json`, etc do not support registry aliases nor prefixes)

## Rationale

Utilizing an existing specification / libraries enables Helm immediate build upon an existing standard.
Rather than Helm inventing its own convention.

`registries.conf` was picked as being a format intended for consumption beyond the Docker application container ecosystem.

## Specification

Helm will utilize the `registries.conf` specification when determinging OCI registry information (authentication credentials, etc):
<https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md>


`registries.conf` will be preferred, either when a `registries.conf` file already exists on the users system.
Or when Helm supports and a user utilizes functionality that can not be supported by Docker's comnfigurtation file in the future.

<!-- The usage of `registries.conf` will be forced, when in the future -->

For example, a registry login command:
```bash
helm registry login "oci.example.com" --username foo --password bar
```

Will result in the configration exerpt (if, and only if, `registries.conf` exists on the users local system):
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

Helm will use the package TBD for updating (and reading) `registries.conf`.

If when reading a registries configation from `registries.conf` results in known configuration Helm doesn't support, Helm must error.
e.g. `location` or non-empty URI path in `prefix`.

An error reading `registries.conf` must result in an error for the user.
Otherwise, users who expect configration from `registries.conf` to be effected, will have a unspected fallback.

To account for existing configration in Docker’s registry configuration, Helm will prefer `registries.conf` when resolving OCI registries (including credentials).
And fall back to the existing store mechanism if `registries.conf` does not contain an entry for the required OCI registry (including if `registries.conf` does not exist).

Helm must expect (and even encourage) users to utilize other tooling to manage `registries.conf`.

## Backwards compatibility

Helm's fallback to Docker's registry configuration ensures the vast majority of existing user workflows remain the same.

However, there are three potential incompatibility scenarios:
2. A corrupt `registries.conf` will cause an error for existing workflows
1. An invalid or incompatible with Helm `registries.conf` entry for the given OCI registry will cause the users workflow to fail
3. Helm’s preference for `registries.conf` will break users who assume credentials are stored in Docker’s registry configuration

The first two can mitigated by users ensuring their systems `registries.conf` is valid, and only includes configuration options Helm supports for the registries they plan to use with Helm.

The last is mitigated by not using `registries.conf` initially unless it exists on the users system.

## Security implications

`registries.conf` introduces a new mechanism for storing OCI credentials, which may introduce credential management vulnerabilities specific to `registries.conf` / `auth.json`

Transitive dependencies of the TBD package for managing registries.conf may introduce security scanner noise (which tends to be a problem in the container library ecosystem)

## How to teach this

Helm's documentation will need to be updated with details of Helm's `registries.conf` support, and fallback to Docker config.

## Reference implementation

<!--
Link to any existing implementation and details about its state, e.g.
proof-of-concept.
-->

## Rejected ideas

### Falling back to Docker’s config upon error reading registries.conf

Falling back upon error means users who expect their configuration to be taken from `registries.conf` will unexpectedly have OCI configuration taken from Docker.
Potentially resulting in difficult to diagnose failures authenticating to the repository.

Rather than "failing fast" and requring users to ensure their configuration to be valid.

## Open issues

- Support for registies.conf in ORAS
- Disucss Helm's/ORAS's usage of registries.conf with owners

## References

### ORAS credential store (registry configuration) docs

https://pkg.go.dev/oras.land/oras-go/v2/registry/remote/credentials#NewStoreFromDocker

### registries.conf specification
https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md
