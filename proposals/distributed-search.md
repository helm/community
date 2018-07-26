# Proposal: Search of Distributed Repositories

Repositories in Helm were designed to be distributed. Instead of a central repository, the concept was for many distributed repositories to exist. This was modeled after [Debian APT](https://en.wikipedia.org/wiki/APT_(Debian)).

To kick start charts, the charts repository was created. Eventually there were the stable and incubator repositories with Helm automatically adding the stable repository. To visually view repositories the monocular project was created which was used to launch what is now hub.kubeapps.com. This site enabled the search of the stable and incubator repositories. With Helm's use of stable repository and the UI search focusing on the community provided charts the community as a whole moved towards a more centralized approach. When companies or others wanted to share their projects they put them into the community charts.

This proposal is about enabling distributed repositories by making them and their packages discoverable.

## A Single Search Location

The goal of this is to have a search site and API that enables the search of many public repositories. Private repositories are a separate scope and can operate with existing tools. Hosting of these repositories is also a non-goal.

The public repositories would need to provide:

1. Valid repositories to the repository specification
1. A security point of contact for the repository
1. A general point of contact for issues
1. Some level of automation around handling the quality of the packages. This is still to be determined and there are more details in the tooling section below.

End users would then be able to query, via the web or API, for charts with an experience containing the following characteristics:

* Can perform a general text search for packages that queries across all listed repositories
* In the details of the response on a chart there will be directions on:

    * Adding the repository to a local helm installation
    * Installing the chart without adding the repository

* Metadata on repositories, charts, and provider organizations.

The idea is that organizations do not need to wait on the community charts team to publish releases or use that process. Instead, package hosting can happen in a distributed manner while the packages can be discoverable.

_Note, under the current more centralized approach there is a level of trust with the charts due to the CNCF oversight and known team handling the day to day activities. While this model loosens and distributes trust we have a goal of trying to keep trust high._

## Tools Provided For Repository Maintainers

To enable distributed repositories the Helm project will provide the following tools:

### Repository Tools

Basic tools needed to run a repository along with pointers to 3rd party services and applications that can be used instead. For example, Codefresh chart repositories, powered by Chartmuseum, and Artifactory would be shared as options.

### Documentation

In addition to tooling there needs to be documentation that covers:

* How to run a repository on GitHub, Gitlab, and other similar code hosting services
* Details on signing charts and how to integrate that into workflows
* Suggestions on leveraging CI as part of repository hosting
* A clearly documented repository specification

### Analysis Tools

To help, and possibly enforce, quality of charts we need to provide tools that can perform an analysis of charts to help validate quality. These tools exist for the stable and incubator charts today. They will be packaged in a manner others can consume and leverage within their workflows.

Note, work on this step has already begun and [can be found on GitHub](https://github.com/kubernetes-helm/chart-testing).

## Outstanding Actions

The following are outstanding actions that need to be worked out but can happen after the proposal is accepted:

* [ ] Decide on and document the requirements for listed repositories

## Organizational Management

The Charts Maintainers team will be the directly responsible party for this system and it will be hosted by the Helm project.