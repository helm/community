# Helm Repo and Container Registry Convergence Proposal

As Helm Charts become the way customers deploy and update multi-image deployments to Kubernetes, which references images stored in a registry, theirs an opportunity to align Helm concepts with registry concepts. 

This proposal suggests aligning some of the commands and chart references to align more consistently with a container registry. 

The proposal doesn't suggest changes in how charts are deployed, rather the storage and referencing of Helm charts.

The proposal covers the following:

1. [Container Registries are collections of Helm Repositories](./001-repo-registry.md)  
  Evolution of [Docker Distribution](https://github.com/docker/distribution) to support Helm Repositories
2. [Helm Charts Align With Registry Repositories](./002-chart-registry-repo.md)  
    Consistent with an image repository, a chart repository is a specific chart, with unique tags representing a versioned history  
3. [Helm Charts use registry authentication](./003-chart-authentication.md)  
4. [`helm fetch` becomes `helm pull`](./004-helm-fetch-pull.md)  
  aligning the commands more consistently
5. [`helm update` & `helm install` support remote urls](./005-helm-fetch-url.md)  
  Without having to set a `--repo` parameter
6. [Helm Chart Versions merge with docker tags](./006-helm-version-tags.md)   
    With all the good and evil of "latest" and "stable" tags and versions.
7. [Local Helm Store becomes a cache](./007-helm-store-cache.md)  
  aligning more with docker run, intermittently does pulls, avoiding the need to continually call `helm repo add`
8. [`helm search` becomes a server side query](./008-helm-search-server-query.md)  
  Further converging the local Helm store as a remote cache

## Background
Helm Charts evolved as a means to streamline and empower deployment of Kubernetes templates. Chart Museum evolved Helm Charts to provide a storage means for Charts. To maximize the ease for server side support, Chart Museum supported minimal storage requirements. Chart Museum can be stored directly in an S3 storage account. As cloud providers implement Helm Repositories, each are facing the same challenges providing charts at cloud-scale. 

## Why Unify?

To be massively successful, Helm and Helm Repositories should appeal and be easily adopted by the masses. Docker has done a great job providing a progressive disclosure of complexity to the user, with a set of commands that most can grok.

The more things are different, the more difficult it is for users to adopt. Helm, Repositories even Kubernetes are the underlying infrastructure we hope all will use. But, we need to leave room for mental processing of concepts to all the other things developers and operational users need to account for.

By converging some of the underlying implementations, along with the commands, we can simplify the number of things users need to remember.

- `helm fetch` vs. 
`docker pull`
- A **Helm Repo** is a collection of charts
- A **Helm Chart** has a collection of versions
- A **container repo** is a **single image**, with a **collection of versions** (tags)
- `docker run` will implicitly issue `docker pull` if the image doesn't exist
- `helm upgrade` requires a `helm fetch` first, or a `--repo` parameter that is different than a image


## Why should Helm change, when "it's better"?

We can certainly make the argument that several of the helm concepts, such as semantic versioning is far better than the open field of tags. And, some of those concepts are preserved, while still aligned with tags. However, Docker has a large installed base, which is only growing. Attempting to either change docker, or insist the changes are justified only increases friction to adoption. 

## Involved Participants

As the goal is broad adoption, we are proposing this proposal be supported by all major clouds ([Azure](www.azure.com), [Google Cloud](https://cloud.google.com/) & [AWS](https://aws.amazon.com)) as well as container vendors, such as [Docker](https://www.docker.com), [Codefresh](https://codefresh.io), [JFrog](https://jfrog.com) and others...

## Proposed Timeline

Helm 3.0 is entering final design phases. Given the growth and quick adoption, with the significance of changes, it's proposed these changes are incorporated into Helm 3.0. 
