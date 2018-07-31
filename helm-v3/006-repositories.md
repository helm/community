# Repositories

The following changes will be made to the repository subsystem:

- A new `helm login` command will be added for authenticating against a repository
- A new `helm push` command will be added for pushing charts to a repository
- The `helm serve` command will be removed
- [ChartMuseum](https://github.com/helm/chartmuseum) will be the Helm subproject
  dedicated to hosting your own repository
- [Monocular](https://github.com/helm/monocular) will be the Helm subproject
  dedicated to distributed repo discovery, and the software powering
  [Hub](https://github.com/helm/hub)

## Authenticating Against A Repository

*In progress. Please see [chartmuseum#59](https://github.com/helm/chartmuseum/issues/59).*

> [name=Matt Farina] There very well may be additional work here around
> registry search and pluggable or otherwise extendable authenication
> mechanisms.

## Pushing Charts To A Repository

Several registries have emerged with support for Helm charts including
ChartMuseum and Quay (via Application Registries) along with extras built on
top of other systems, such as object storage (e.g., see the [S3
Plugin](https://github.com/hypnoglow/helm-s3)). These registries are asking for
and could benefit from a capability to push packages from the Helm client.

To support registries there will be a `helm push` command that works in a
similar manner to `helm fetch`. It will support different systems via the
URI/URL scheme. It will provide a default implementation for HTTP(S) based
on the ChartMuseum API spec.

Other systems, such as S3, can implement an uploader via a plugin for schemes Helm does not
provide an implementation for.

The [helm-push plugin](https://github.com/chartmuseum/helm-push) describes potential semantics
for a `helm push` command.
Here are some usage examples:
 ```
# push .tgz from "helm package"
helm push mychart-0.1.0.tgz myrepo

# package and push chart directory
helm push mychart/ myrepo

# override version in Chart.yaml
helm push mychart/ --version="0.2.0-rc1" myrepo

# push directly to chart repo URL
helm push mychart/ https://my.chart.repo.com
```
