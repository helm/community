# Proposal 1 - Container Registries provide Helm Repositories

As Helm Charts are designed around the intention of deploying images, it's a natural evolution to store Helm Charts in registries, alongside the images they reference. By aligning charts within a registry, registry vendors can extend the auth infrastructure to provide a common means to authenticate charts and the referenced images, as well as provide a common UI for search, discovery and additional meta-data. 

As Helm is evolving as the defacto deployment technology, it's believed it's time to evolve registries to support additional artifacts, such as Helm Charts.


For registries to support Helm Charts, a few changes are proposed
1. [Helm Charts align with registry repositories](./002-chart-registry-repo.md)  
  enabling fully qualified charts
1. [Helm Charts use registry authentication](./003-chart-authentication.md)  
1. [Helm Chart Versions merge with docker tags](./006-helm-version-tags.md)   
    With all the good and evil of "latest" and "stable" tags and versions.
1. [Local Helm Store becomes a cache](./007-helm-store-cache.md)  
  aligning more with docker run, intermittently does pulls, avoiding the need to continually call `helm repo add`
1. [`helm search` becomes a server side query](./008-helm-search-server-query.md)  
  Further converging the local Helm store as a remote cache


## Converging with Registries
While docker provides a baseline reference implementation through [docker distribution](https://github.com/docker/distribution), cloud providers of registries have had to fill in specifics for high scale/concurrent delivery. 

Delivering images at cloud scale requires a number of enhancements including:
- indexing
- caching
- authentication
- versioning

The evolution and implementation required to deliver Image Manifests and images at cloud scale are a super-set of the needs to deliver charts at cloud scale. 

## Upstream Support to Docker Distribution 

Proposal to commit upstream changes to [docker distribution](https://github.com/docker/distribution) to enable all implementors of the upstream project to easily adopt Helm Repositories. 
