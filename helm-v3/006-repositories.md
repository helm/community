# Repositories

## Current State
Helm v2 originally shipped with a model for managing remote repositories of Charts analogous to the model used for package managers used by Linux distributions (e.g. APT for Debian/Ubuntu). As the design for Helm v3 evolves, it is vital to reflect on where this model has provided value and where it has fallen short.

### Pros

- Ubiquity
  - Because they shipped natively with the Helm client, every Helm user had easy access officially packaged Charts.
- Familiarity
  - Because most Helm users have used Linux distributions, there is small learning curve to understanding the Helm repository model.
- Simplicity
  - No reliance on server-side infrastructure, option to store static index.yaml and .tgz packages in an S3 bucket, etc.
  - Helm Chart distributors, even for internal company packages, could create repositories without any additional software.
- Security
  - Search for packages happens locally enabling the search of more than one repository without the risk of leaking information to a repository that should not see the information (information disclosure issue).
  - Security models for repositories could use existing vetted systems such as object storage that already meet compliance requirements.

### Cons

- Client-side repository indexes
  - Because the client performs actions locally on a cached index, larger repositories have performance issues.
  - https://github.com/kubernetes/helm/issues/3557 (Feb 22, 2018)
	
		> [name=Matt Farina] Note, in issue #3557 it notes that the issue of memory has to do with how the index file is being loaded into memory and being worked with. Is the issue a client-side index one (as an option) or how it is implemented in helm v2? For example, what if the data was stored in an sqlite database in the helm home directory and that were queried? Would the memory footprint be different? I'm asking to flush out the root issue.
		> > [name=Jimmy] For perspective, larger container registry installations are hundreds of gigabytes of metadata in the database.
		> > > [name=Matt Farina] Have you looked at how launchpad.net does it for APT repos?
- Manual upload of index.yaml
  - https://github.com/kubernetes/helm/issues/1896 (Feb 1, 2017)
  - Subject to race conditions
  - This is the core problem that ChartMuseum solves
- No client capability for server-side mutations
  - Because the clients can only perform read actions, there is no good solution for managing the creation and updating of Charts.
  - https://github.com/kubernetes/helm/issues/3564
  - ChartMuseum and App Registry provide capability, but this "push" operation not native to Helm CLI
  - Note, the [Helm S3 plugin](https://github.com/hypnoglow/helm-s3) has push capability which would be useful to carry forward, even if still a plugin.
- Unique repository model
  - Because there was no pre-existing server-side software that could be reused, it is taking a long time for server-side software (e.g. App Registry, ChartMuseum) to mature.
  - https://github.com/kubernetes-helm/chartmuseum/pull/53

		> [name=Matt Farina] How is the model unique for a package manager? It's not "docker-like" but it does mirror APT which is a package manager? Did you mean something else by the top level bullet "Unique repository model"?
		> > [name=Jimmy Zelinskie] I literally just meant that it isn't something that already exists in an off-the-shelf form. The original goal was that you could use any webserver to serve Charts, however software had to be written to upload those Charts and manage the remote index.
		> > > [name=Matt Farina] I wonder if this should be a PRO rather than a CON. Because there was no registry service many people were able to fit it into their existing workflows. For example, I used to work on a team that had artifacts which were uploaded to a web served and fetched from there. Helm artifacts could be placed alongside other ones in a different directory. No new service and we already had upload tooling.
- Authentication
  - Only HTTP basic auth, client-side TLS supported
  - Please see https://github.com/kubernetes/helm/issues/1038 (Aug 10, 2016)

## Helm 3 Requirements

Having now reflected on the previous design, there should be a list of requirements going into the design of the new system.

- Should not preclude the ability for Helm to execute Helm v2 Charts
- Should be decentralized without sacrificing a user experience where it is easy to search and discover useful Charts across the decentralized infrastructure.
- Should explicitly define a protocol for remotely managing and discovering Charts
	
	> [name=Matt Farina] Do you have examples of a cases that do this with good and easy (for end users like application operator) security practices? Note, Dockers Hub has accidential information disclosures to it that we need to try and protect against.
	> > [name=Jimmy] That example doesn't have anything to do with the protocol, but everything to do with the user-experience of the CLI client. This bullet point was more or less just outlining that there should be a well-defined specification for the protocol for developers of registries/clients.
	> > > [name=Matt Farina] A specification does make some assumptions. Is there a way to have a protocol for remote discovery, for your definition of it, in a staticly generated registry? We may have different ideas of what's going on here.
	
- Should support single-tenant and multi-tenant workflows

	> [name=Matt Farina] What would change about the spec/interface Helm knows to support multi-tenant?
	
- Should provide an intuitive user-experience (e.g. docker-like)

	> [name=Matt Farina] If helm had a `helm push` command that worked in a similar way to the existing `helm fetch` command wouldn't that provide for the "docker-like" need? Note, helm plugins can own URI scheme and make fetch work for that today. For example, the [Helm s3 plugin](https://github.com/hypnoglow/helm-s3) lets you have private repos in S3 today. If a URI for `s3://` is used in a chart `helm fetch` or commands that use the internals of this know to route through the plugin.
	> > [name=Jimmy Zelinskie] I need to look into this, because I remember when we built the App Registry plugin, the support was not sufficient especially not for dependency resolution.
	> > > [name=Matt Farina] The plugin download by scheme also works for dependencies listed in requirements files.

	> [name=Matt Farina] And, you can `helm fetch` without first adding a repository by using the full URI.
	
- Should follow best practices for authentication (e.g. OAuth2)

	> [name=Matt Farina] It should be possible to support both basic and OAuth2 out of the box. Even drop in to a plugin if an auth type is unrecognized. What about that form of model?
	> > [name=Josh Dolitsky] please see ChartMuseum Auth proposal: https://github.com/kubernetes-helm/chartmuseum/issues/59
	
- The security model MUST be acceptable for enterprise and compliance (e.g., [SOX](https://en.wikipedia.org/wiki/Sarbanes%E2%80%93Oxley_Act)) situations.
	
	> [name=Jimmy Zelinskie] oh boy, SOX 😱
	> > [name=Matt Farina] Having suffered through compliance issues, I've learned to dot the i's and cross those t's. Sometimes they don't even make good sense. Sometimes it's a good thing we're forced to do them. Compliance is one of the things that makes object storage appealing. The backup/restore, logging, auth[nz] models are known.

### Important Links

- [Previous Hosted Chart Repository Reqs Gist](https://gist.github.com/mattfarina/9f9ebe1171943e873576c0ea3e6f4f30)


## Helm 3 Proposal

A new API version for chart repositories (**v2**) will be introduced to provide new functionality address issues of scale (see [#3557](https://github.com/kubernetes/helm/issues/3557)).

The following changes will be made to the repository subsystem:

- v2 will be the new default
- v1 will be supported in its current state, but will not benefit from new functionality
- The `helm search` command will become a server-side operation for v2 repos
- No reliance on local index for `helm fetch` `helm inspect` `helm install` etc for v2 repos, packages are at known location in repo
- `~/.helm/config` will be introduced (overridden by `HELMCONFIG` env var) which contains sensitive information necessary to authenticate against a v2 repo, with a default context
- A new `helm config` command with a `set-context` subcommand will be added in to switch contexts defined in `~/.helm/config`
- A new `helm push` verb will be added for pushing charts to a repository
- `helm repo index` and `helm repo update` will only be used for v1 repos, and flagged as "deprecated"
- The `helm serve` command will be removed
- Thorough docs on how to run a private [ChartMuseum](https://github.com/kubernetes-helm/chartmuseum) server will be provided

### Searching a Repository
*TODO: notes on usage of `helm search` and new API spec for search/fetch, where chart packages exist at known locations*

### Authenticating Against a Repository
*TODO: notes on usage of `helm config` and format of `~/.helm/config`*

### Pushing Charts to a Repository
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