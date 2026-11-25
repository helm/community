# Helm Governance

The following document outlines how the Helm project governance operates.
## The Helm Project

The Helm project is made up of several codebases and services with different release cycles that enable users to create, distribute, and operate packages in Kubernetes. These codebases include:

* Helm - the package manager
* Charts - community curated charts
* Chartmuseum - a chart repository with support for pushing charts, auth, and more
* chart-testing - container based chart testing tools

The services provided include:
* Documentation for those who want to create, distribute, depend on, and operate packages
* Public chart collaboration
## Maintainers Structure

Helm has a single level of maintainership.
The Helm maintainers collectively oversee the overall project and its health.
Maintainers are organized into groups responsible for one or more codebases, services, or other project activities.
Technical responsibilities and permissions are scoped to the areas owned by each maintainer group.

Maintainer groups, their membership, and associated repositories are documented
in [maintainer-groups.yaml](../maintainer-groups.yaml).

Changes in maintainership have to be announced on the [Helm mailing list](https://lists.cncf.io/g/cncf-helm).
### Project-wide Responsibilities

The Helm maintainers are collectively responsible for:
* Maintaining the mission, vision, values, and scope of the project
* Refining the governance and charter as needed
* Making project level decisions
* Resolving escalated project decisions when the subteam responsible is blocked
* Managing the Helm brand
* Controlling access to Helm assets such as source repositories, hosting, project calendars
* Handling code of conduct violations
* Deciding what sub-groups are part of the Helm project
* Overseeing the resolution and disclosure of security issues
* Managing financial decisions related to the project

No one company or organization can employ a simple majority of the Helm maintainers.

The Helm maintainers will select a chair to set agendas and call maintainer meetings.
### Maintainer Groups

Maintainers are responsible for activities surrounding the development and release of code,
the operation of any services they own (e.g., the documentation site),
or other tasks within their areas of ownership.
Technical decisions reside with the maintainers of the affected groups.
Decisions spanning multiple maintainer groups that cannot be resolved by those groups
can be escalated to the Helm maintainers collectively.

To be considered active maintainers, maintainers must be active members of at least one maintainer group.
Maintainers who are no longer members of any active maintainer group become emeritus maintainers.
Maintainers do not need to be software developers. No explicit role is placed upon them and they can be anyone appropriate for the work being produced. For example, if a repository is for documentation it would be appropriate for maintainers to be editors.

Changes to maintainers use the following:
* A maintainer may step down by emailing the [Helm mailing list](https://lists.cncf.io/g/cncf-helm)
* Maintainers MUST remain active. If they are unresponsive for > 3 months they will be automatically removed unless a [super-majority](https://en.wikipedia.org/wiki/Supermajority#Two-thirds_vote) of the other maintainers of the affected group agrees to extend the period to be greater than 3 months
* New maintainers can be added to a maintainer group by a [super-majority](https://en.wikipedia.org/wiki/Supermajority#Two-thirds_vote) vote of the existing maintainers of that group. Nomination can happen on the public [Helm mailing list](https://lists.cncf.io/g/cncf-helm) with voting on the private maintainer list or it can happen from a pull request to the appropriate record of maintainership for the group (e.g., OWNERS file) with voting happening through approvals on the pull request.
* While nomination will happen on the public [Helm mailing list](https://lists.cncf.io/g/cncf-helm), voting will happen on the private maintainer list.
* When a project has no maintainers the Helm maintainers collectively become responsible for it and may archive the project or find new maintainers
* When a maintainer group is retired, its maintainers become emeritus maintainers unless they belong to another active maintainer group.

### Delegated Teams

The Helm maintainers may delegate specific responsibilities and associated permissions to teams of active Helm maintainers.
Membership in a delegated team does not confer maintainership or additional governance authority.
Members who cease to be active Helm maintainers are no longer eligible to serve on delegated teams.

The Helm Admins and [Security Team](../SECURITY.md) are delegated teams.

### Helm Admins

Helm Admins administer shared Helm project resources, including the GitHub organization and associated infrastructure.
The [helm/community](https://github.com/helm/community) repository is administered by the Helm Admins
on behalf of the Helm maintainers.
Changes to governance and other project-wide matters are subject to the decision-making process defined in this document.
They review and approve changes to designated governance and project-level files,
ensuring that any required decision has been made according to this document before approving those changes.
Their approval does not substitute for any required vote of the maintainers.
The Helm Admin role does not confer additional decision-making authority.

Helm Admins are appointed and removed by a super-majority vote of the Helm maintainers.
They can be contacted at helm-admin@lists.cncf.io.

## Decision Making

### Deciders

* Decisions within the scope of a maintainer group's responsibilities are made by the maintainers of that group.
* Project-wide decisions and escalated decisions are made by all Helm maintainers.

Each maintainer has one vote in decisions for which they are eligible, regardless of the number of maintainer groups they belong to.

### Decision Making Process

When maintainers need to make decisions there are two ways decisions are made, unless described elsewhere.

The default decision making process is [lazy-consensus](http://communitymgt.wikia.com/wiki/Lazy_consensus). This means that any decision is considered supported by the team making it as long as no one objects. Silence on any consensus decision is implicit agreement and equivalent to explicit agreement. Explicit agreement may be stated at will.
When a consensus cannot be found a maintainer can call for a [majority](https://en.wikipedia.org/wiki/Majority) vote on a decision.

Many of the day-to-day project maintenance can be done by a lazy consensus model. But the following items must be called to a project-wide vote:
* Enforcing a CoC violation (super majority)
* Removing a maintainer for any reason other than inactivity (super majority)
* Changing the governance rules (this document) (super majority)
* Licensing and intellectual property changes (including new logos, wordmarks) (simple majority)
* Adding, archiving, or removing subprojects (simple majority)
* Utilizing Helm/CNCF money for anything CNCF deems "not cheap and easy" (simple majority)
Changes to governance must receive the required super-majority approval before being merged.

Other decisions may, but do not need to be, called out and put up for decision on the [Helm mailing list](https://lists.cncf.io/g/cncf-helm) at any time and by anyone. By default, any decisions called to a vote will be for a _simple majority_ vote.
## Code of Conduct

The code of conduct is overseen by the Helm maintainers. Possible code of conduct violations should be emailed to the maintainers at cncf-helm-maintainers@lists.cncf.io.

If the possible violation is against one of the maintainers that member will be recused from voting on the issue. Such issues must be escalated to the appropriate CNCF contact, and CNCF may choose to intervene.
## DCO and Licenses

The following licenses and contributor agreements will be used for Helm projects:

* [Apache 2.0](https://opensource.org/licenses/Apache-2.0) for code
* [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode) for documentation
* [Developer Certificate of Origin](https://developercertificate.org/) for new contributions
