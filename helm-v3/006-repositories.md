# Repositories

A new API version for chart repositories (**v2**) will be introduced to provide new functionality address issues of scale (see [#3557](https://github.com/kubernetes/helm/issues/3557)).

The following changes will be made to the repository subsystem:

- v2 will be the new default
- v1 will be supported in its current state, but will not benefit from new functionality
- The `helm search` command will become a server-side operation for v2 repos
- No reliance on local index for `helm fetch` `helm inspect` `helm install` etc for v2 repos
- `~/.helm/config` will be introduced (overridden by `HELMCONFIG` env var) which contains sensitive information necessary to authenticate against a v2 repo, with a default context
- A new `helm config` command with a `set-context` subcommand will be added in to switch contexts defined in `~/.helm/config`
- A new `helm push` verb will be added for pushing charts to a repository
- `helm repo index` and `helm repo update` will only be used for v1 repos, and flagged as "deprecated"
- The `helm serve` command will be removed
- Thorough docs on how to run a private [ChartMuseum](https://github.com/kubernetes-helm/chartmuseum) server will be provided

## Searching a Repository
*TODO: notes on usage of `helm search` and new API spec for search/fetch, where chart packages exist at known locations*

## Authenticating Against a Repository
*TODO: notes on usage of `helm config` and format of `~/.helm/config`*

## Pushing Charts to a Repository
*TODO: notes on usage of `helm push`*

Several registries have emerged with support for Helm charts including
ChartMuseum and Quay (via Application Registries) along with extras built on
top of other systems, such as object storage (e.g., see the [S3
Plugin](https://github.com/hypnoglow/helm-s3)). These registries are asking for
and could benefit from a capability to push packages from the Helm client.

To support registries there will be a `helm push` command that works in a
similar manner to `helm fetch`. It will support different systems via the
URI/URL scheme and provide a default implementation for HTTP(S). Other systems,
such as S3, can implement an uploader via a plugin for schemes Helm does not
provide an implementation for.

> [name=Matt Farina] There very well may be additional work here around
> registry search and pluggable or otherwise extendable authenication
> mechanisms.
> 
> [name=Adnan Abdulhussein] What would the default HTTP(S) implementation be?
> [name=Matt Fisher] Adnan, would you mind clarifying your comment a little?
> 
> [name=Adnan Abdulhussein] Sorry, the proposal mentions there will be a
> default implemention for `helm push` to a regular HTTP(S) chart repository.
> It's unclear to me how this would actually work.
> 
> [name=Adnan Abdulhussein] Sounds like this is something that will be fleshed
> out more in Josh's proposal, so I'll wait for that :)
