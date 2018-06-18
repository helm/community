# Repositories

The following changes will be made to the repository subsystem:

- The `helm serve` command will be removed
- A new `helm push` verb will be added for pushing charts to a repository
- The index file will be JSON instead of YAML

> This document is in-progress and needs input from Chart Museum.

## Pushing Charts To A Repository

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

## JSON Index File

Helm v2 leveraged YAML for the index of packages and package versions shared
between Helm and a repository. Helm v3 will move to a version 2 of the
repository specification and will use JSON instead of YAML. In place of
`index.yaml` a repository will serve an `index.json` file.

The reason for the format change is due to the performance of software that
interacts with these formats. For example, YAML has anchors and aliases that
parsers and serializers need to interact with. This causes parsers to use more
memory and have slower performance. The Helm use of indexes does not leverage
these features.

For compatibility, it will be possible to serve both an `index.yaml` file and an
`index.json` file. This will allow a repository to support both Helm versions 2
and 3. Helm v3 will provide the ability to generate both an `index.json` and
`index.yaml` file to enable backwards compatibility.

In addition to generating the legacy `index.yaml` file, when Helm queries a
repository and does not find an `index.json` file it will attempt to use the
legacy `index.yaml` format.