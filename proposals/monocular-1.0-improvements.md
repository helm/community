# Proposal: Monocular 1.0 - improvements and redefining project focus

Monocular was created to provide a simple interface to browse and discover charts in the stable and incubator community repositories, and a running instance of Monocular is setup at hub.kubeapps.com (originally kubeapps.com).

As the Charts project moves to a [more distributed repository model](https://github.com/helm/community/blob/master/proposals/distributed-search.md), an instance of Monocular will be available at hub.helm.sh to serve as a home for community-contributed charts from multiple repositories.

The current codebase has outgrown the initial requirements for the Monocular project, and this proposal is about making Monocular more robust and scalable, and redefining the focus of the project to suit the needs of the CNCF Helm Hub.

## Replacing the Monocular API server

The current Monocular API server is a monolithic Go-based server which handles the periodic syncing and ingestion of charts, and serves a REST API to read chart data, manage configured repositories and handle in-cluster chart installations and management. This architecture has shown issues with scalability and flexibility that can be improved by splitting services out into separate components.

The [Kubeapps project](https://github.com/kubeapps/kubeapps) includes components that will be contributed back to Monocular to achieve this.

### Storing chart metadata in a database

In the current Monocular implementation, chart metadata is cached in memory. This has the requirement for repository syncing to be part of the API server process and also presents issues when scaling out the API server as each process holds an independent cache of the chart metadata.

This will be changed to storing chart metadata in a database, so that each API server instance can retrieve the chart metadata from the same consistent store. MongoDB will be used to store the chart metadata as the NoSQL model makes it easy to dump chart metadata directly from repository indexes.

### Chartsvc

[Chartsvc](https://github.com/kubeapps/kubeapps/tree/master/cmd/chartsvc) is a small Go API server built to read and serve chart information from a MongoDB database. This service will replace the existing API server for fetching metadata about charts.

### Syncing chart repositories

One of the biggest scalability issues in Monocular today is the implementation for syncing chart repositories in the API server. Since the syncing is tied to the running of the API server, it is difficult to manage and debug when something goes wrong during the sync process. Bugs and issues can lead to the API server crashing. It is also not possible to manually trigger a repository sync.

The [chart-repo](https://github.com/kubeapps/kubeapps/tree/master/cmd/chart-repo) tool can be run as a background job to populate a MongoDB database with chart metadata from a particular chart repository. Each configured chart repository will have a separate background job run periodically to sync the chart metadata.

[The AppRepository Kubernetes CRD and controller](https://github.com/kubeapps/kubeapps/tree/master/cmd/apprepository-controller) will be used to manage the configured chart repositories and create [Kubernetes CronJobs and Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) to orchestrate the background jobs to run the chart-repo tool described above. This will make Kubernetes the default environment for operating Monocular, though it is possible to replace this component to orchestrate the background jobs in other ways on other platforms.


## Removing in-cluster support

Monocular currently includes a basic implementation of in-cluster application management. If Monocular is deployed within a Kubernetes cluster, a feature flag can be enabled to allow the installation of charts directly through the UI.

Providing a good solution for deploying and managing apps in-cluster is an orthogonal user experience to a public search and discovery site. There is other tooling that can support this usecase better (e.g. Helm CLI, [Kubeapps](https://github.com/kubeapps/kubeapps), [RedHat Automation Broker](https://blog.openshift.com/automation-broker-discovering-helm-charts/)).

To focus on the CNCF Helm Hub user experience, support for in-cluster features will be dropped. A v0.7 (the current Monocular version) branch will be created prior to these changes so that this functionality can be used and supported for some time until users have fully migrated to another tool.
