# Helm Governance

The Helm governance details areas such as:

* Maintainer responsibilities
* How maintainers are organized into projects and groups
* How group members are selected
* Decision making
* Code of Conduct handling

For the full detail on the Helm governance see the document [governance.md](governance.md).

Helm has a single level of maintainer. The projects, the groups, and where each one records its members are listed in [the community repo Maintainers file](../MAINTAINERS.md).

## Projects

The Helm governance refers to projects for the Helm project. The following Helm
projects currently exist:

* **Helm Core**: The Helm CLI and packages
* **Charts**: Responsible for chart-related tooling (e.g., chart testing)
* **ChartMuseum**: The chart repository server
* **Web and Docs**: The Helm websites
* **Mapkubeapis plugin**: The plugin that updates release metadata for removed Kubernetes APIs
* **Community Management**: Community programs and events

All other repositories are either archived or owned by the Helm maintainers as a whole.

## Groups

Groups are subsets of the maintainers with a specific duty. The following groups
currently exist:

* **GitHub Admin**: Holds the owner role on the Helm GitHub organization
* **Security Team**: Responds to vulnerability reports as described in [SECURITY.md](../SECURITY.md)
