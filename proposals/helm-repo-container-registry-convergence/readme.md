# Helm Repos and Container Registry Convergence Proposal

As Helm Charts become the way customers deploy and update multi-image deployments to Kubernetes, which references images stored in a registry, theirs an opportunity to align Helm concepts with registry concepts. 

This proposal suggests aligning some of the commands and chart references to align more consistently with a container registry. 

The proposal doesn't suggest changes in how charts are deployed, rather the storage and referencing of Helm charts.

While Helm integration can be implemented uniquely by IVS/Cloud providers, such as Azure, Quay and Harbor, the primary purpose of this spec is to integrate registries into the core **Helm** CLI. By moving registry capabilities into the Helm client, all Helm users will have a consistent & productive experience as they work with multiple cloud providers. 

The proposal covers the following user experience:

- `helm login demo42.azurecr.io -u $USER -p $PWD`
- `helm push ./superbowl demo42.azurecr.io/marketing/shoes/superbowl:1.0`
- `helm pull demo42.azurecr.io/marketing/shoes/superbowl:1.0`
- `helm upgrade superbowl-campaign demo42.azurecr.io/marketing/shoes/superbowl:1.0 --reuse values --set webui=demo42.azurecr.io/marketing/shoes/superbowl:a3nf`

  **Note:** `helm upgrade ...` can be executed without `helm pull`

- `helm search demo42.azurecr.io superbowl --versionFrom 1.0 --versionTo 2.1`

  **Note:** likewise, `helm push` and `helm upgrade` can be performed without `helm search`. Search becomes a discovery api to aid the user for which charts to pull/install

The proposal is broken down into the follow sub sections: 

1. [Container Registries are collections of Helm Repositories](./001-repo-registry.md)  
  Evolution of [Docker Distribution](https://github.com/docker/distribution) to support Helm Repositories
2. [Helm Charts Align With Registry Repositories](./002-chart-registry-repo.md)  
    Consistent with an image repository, a chart repository is a specific chart, with unique tags representing a versioned history  
3. [Helm Charts use registry authentication](./003-chart-authentication.md)  
4. [~~`helm fetch` becomes `helm pull`~~](./004-helm-fetch-pull.md)  
  aligning the commands more consistently
5. [`helm update` & `helm install` support remote urls](./005-helm-fetch-url.md)  
  Without having to set a `--repo` parameter
6. [Helm Chart Versions merge with docker tags](./006-helm-version-tags.md)   
    With all the good and evil of "latest" and "stable" tags and versions.
7. [Local Helm Store becomes a cache](./007-helm-store-cache.md)  
  aligning more with docker run, intermittently does pulls, avoiding the need to continually call `helm repo add`
8. [`helm search` becomes a server side query](./008-helm-search-server.md)  
  Further converging the local Helm store as a remote cache
9. [`helm push` encompasses `helm package`](./009-helm-push.md) to avoid the details of having the tar up the files.

## Background
Helm Charts evolved as a means to streamline and empower deployment of Kubernetes templates. Chart Museum evolved Helm Charts to provide a storage means for Charts. To maximize the ease for server side support, Chart Museum supported minimal storage requirements. Chart Museum can be stored directly in an S3 storage account. As cloud providers implement Helm Repositories, each are facing the same challenges providing charts at cloud-scale. 

## Why Unify?

To be massively successful, Helm and Helm Repositories should appeal and be easily adopted by the masses. Docker has done a great job providing a progressive disclosure of complexity to the user, with a set of commands that most can grok.

The more things are different, the more difficult it is for users to adopt. Helm, repositories and kubernetes are the underlying infrastructure we hope all will use. However, we need room for mental processing of concepts to all the other things developers and operational users need to account for.

By converging some of the underlying implementations, along with the commands, we can simplify the number of things users need to remember.

- `helm fetch` vs. 
`docker pull`
- A **Helm Repo** is a collection of charts
- A **Helm Chart** has a collection of versions
- A **Container repo** is a **single image**, with a **collection of versions** (tags)
- `docker run` will implicitly issue `docker pull` if the image doesn't exist
- `helm upgrade` requires a `helm fetch` with `--untar` first, or a `--repo` parameter that is different than a image


## Why should Helm change, when "it's better"?

One can certainly argue that several of the helm concepts, such as semantic versioning is far better than the open field of tags. And, some of those concepts are preserved while still aligned with tags. However, Docker has a large installed base, which is only growing. Attempting to either change docker, or insist the changes are justified only increases friction to adoption. By moving the new concepts, such as immutable tags and channels into OCI, all container artfifact types can benefit.

## Compatibility 

Users have create large collections of Helm 2 artifacts, stored in [Chart Museum](https://chartmuseum.com/), , JFrog, Harbor and [Azure Container Registry](https://aka.ms/acr/helm-repos) backed repos.

Helm 3 registry support will expand the helm community by consolidating Images and Helm Charts into OCI compliant registries. Any OCI Compliant Registry can be a Helm Repository.

Users can more easily update their clients to gain registry support. As such, the following compatibility is proposed:

|Client|Helm 2| Helm 3|
|-|-|-|
| Local Repos | * | * |
|Chart Museum Repos | * | *1
|OCI Backed Registries | N/A | *

1. Chart Museum is looking to add OCI registries as a default option to a future Chart Museum version. The goal is to continue the Chart Museum brand customers have become aquatinted with, as the thing they can easily install and run. 

A Helm 2 client will simply not have knowledge of, or access to the APIs required to access OCI Backed registries.

By comparison, a Helm 3 client will have full backwards compatibility to retrieve charts. 

From a chart retrieval perspective, a Helm user should have no reason to not simply upgrade to a Helm 3 client. 

## Involved Participants

The goal is broad adoption. By shifting registry dependencies to OCI, we hope this proposal will be supported by all major clouds vendors ([Azure](www.azure.com), [AWS](https://aws.amazon.com), [Google Cloud](https://cloud.google.com/), [Quay](https://quay.io)), ISVs ([Docker](https://www.docker.com), [Codefresh](https://codefresh.io), [JFrog](https://jfrog.com)), community projects ([Harbor](https://goharbor.io/)) and any other project/vendor supporting OCI.

## Proposed Timeline

Helm 3.0 is entering final design phases. Given the growth and quick adoption, with the significance of changes, it's proposed these changes are incorporated into Helm 3.0. 
