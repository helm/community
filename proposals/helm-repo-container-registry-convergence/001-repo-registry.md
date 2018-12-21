# Proposal 1 - Container Registries provide Helm Repositories

As Helm Charts are designed around the intention of deploying images, it's a natural evolution to store Helm Charts in registries, alongside the images they reference. By aligning charts within a registry, registry vendors can extend the auth infrastructure to provide a common means to authenticate charts and the referenced images, as well as provide a common UI for search, discovery and additional meta-data. 

As Helm is evolving as the defacto deployment technology, it's believed it's time to evolve registries to support additional artifacts, such as Helm Charts.


For registries to support Helm Charts, a few changes are proposed
- [Proposal 2 - Helm Charts align with registry repositories](./002-chart-registry-repo.md)  
  enabling fully qualified charts
- [Proposal 3 - Helm Charts use registry authentication](./003-chart-authentication.md)  
- [Proposal 6 - Helm Chart Versions merge with docker tags](./006-helm-version-tags.md)   
    With all the good and evil of "latest" and "stable" tags and versions.
- [Proposal 7 - Local Helm Store becomes a cache](./007-helm-store-cache.md)  
  aligning with docker run, avoiding the need to call `helm repo add` or `helm repo update`
- [Proposal 8 - `helm search` becomes a server side query](./008-helm-search-server-query.md)  
  Further aiding discovery


## Converging with Registries
While docker provides a baseline reference implementation through [docker distribution](https://github.com/docker/distribution), cloud providers of registries have had to fill in specifics for high scale/concurrent delivery. 

Delivering images at cloud scale requires a number of enhancements including:
- indexing
- caching
- authentication
- versioning

The evolution and implementation required to deliver Image Manifests and images at cloud scale are a super-set of the needs to deliver charts at cloud scale. 

## Upstream Support to OCI Distribution 

Work has begun with OCI distribution, standardizing registries to store generic artifact types. The OCI working proposal presumes a mimeType is defined, allowing any artifact type to be pushed to a registry. By moving to this model, Helm can use a registry as a generic artifact store, targeting any OCI compliant registry. Assuming all cloud providers move to OCI, a helm user, with the standard `helm` client could push & pull images to **acr**, **docker hub**, **ecr**8*, **gcr**, **harbor**, **quay** and others.
