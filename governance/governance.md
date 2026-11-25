# Helm Governance

The following document outlines how the Helm project governance operates.

## The Helm Project

The Helm project is made up of several codebases and services with different release cycles that enable users to create, distribute, and operate packages in Kubernetes. Each codebase, group of related codebases, or service is a _project_. The projects include:

- Helm - the package manager
- Charts - container based chart testing and release tools (e.g., chart-testing, chart-releaser)
- ChartMuseum - a chart repository with support for pushing charts, auth, and more
- Web and Docs - the Helm websites and documentation for those who want to create, distribute, depend on, and operate packages

The current list of projects, their repositories, and where each one records its maintainers is in [maintainer-groups.yaml](../maintainer-groups.yaml).

## Maintainers Structure

Helm has one level of maintainer. Anyone who maintains at least one active (non-archived) project is a Helm maintainer, and every Helm maintainer has one vote on matters that affect the Helm organization as a whole.

Maintainers are organized in two ways:

- **Projects** scope the day-to-day work. A maintainer is listed on one or more projects and makes the technical decisions for those projects.
- **Groups** are subsets of the maintainers who carry out a specific duty on behalf of all maintainers (e.g., the GitHub Admin group and the security team). A group has only the authority this document or the maintainers give it.

Changes in maintainership have to be announced on the [Helm mailing list](https://lists.cncf.io/g/cncf-helm).

### Helm Maintainers

Together, the Helm maintainers are responsible for:

- Maintaining the mission, vision, values, and scope of the project
- Refining the governance and charter as needed
- Making organization level decisions
- Resolving escalated decisions when the projects involved are blocked
- Managing the Helm brand
- Handling code of conduct violations
- Deciding what projects and groups are part of Helm
- Overseeing the resolution and disclosure of security issues
- Managing financial decisions related to Helm

These decisions are made using the process in [Decision Making at the Helm org level](#decision-making-at-the-helm-org-level).

Maintainers do not need to be software developers. No explicit role is placed upon them and they can be anyone appropriate for the work being produced. For example, if a repository is for documentation it would be appropriate for maintainers to be editors.

To be considered active, a maintainer must be associated with at least one active, non-archived project. A maintainer listed only on archived projects becomes an emeritus maintainer. An emeritus maintainer has no vote and cannot serve in a group.

Triage maintainers, described in [HIP 14](../hips/hip-0014.md), are not Helm maintainers for the purposes of this document. They do not vote.

### Projects

The maintainers of a project are responsible for activities surrounding the development and release of code, the operation of any services the project owns (e.g., the documentation site), or the tasks needed to execute the project (e.g., community management, setting up an event booth). Technical decisions for a project reside with the maintainers of that project. When a decision crosses projects and the maintainers involved cannot resolve it, any of them can escalate it to all Helm maintainers.

Some projects cover more than one repo (e.g., the Charts project manages chart-testing, chart-releaser, and charts-tooling). Others cover a single repo (e.g., ChartMuseum).

Each project keeps a record of its maintainers (e.g., an OWNERS file). Changes to the maintainers of a project use the following:

- A maintainer may step down from a project by emailing the [Helm mailing list](https://lists.cncf.io/g/cncf-helm)
- Maintainers MUST remain active. If they are unresponsive for > 3 months they will be automatically removed from a project unless a [super-majority](https://en.wikipedia.org/wiki/Supermajority#Two-thirds_vote) of the other maintainers of that project agrees to extend the period to be greater than 3 months
- New maintainers can be added to a project by a [super-majority](https://en.wikipedia.org/wiki/Supermajority#Two-thirds_vote) vote of the existing maintainers of that project. Nomination can happen on the public [Helm mailing list](https://lists.cncf.io/g/cncf-helm) with voting on the private maintainer list. It can also happen from a pull request to the record of maintainers for the project with voting happening through approvals on the pull request
- A person added to their first project becomes a Helm maintainer at the same time. No second vote is held
- When a project has no maintainers the Helm maintainers as a whole become responsible for it and may archive the project or find new maintainers
- When maintainers no longer maintain active projects due to project archival or removal, they become emeritus maintainers unless they join another Helm project as maintainers

Repositories that belong to no project (e.g., this community repository) are the responsibility of the Helm maintainers as a whole.

### Groups

A group is a set of Helm maintainers who handle a duty that should sit with a small number of people. A group serves the maintainers. Membership in a group gives no extra vote and no say over the technical direction of any project.

The maintainers can create or dissolve a group with a simple majority vote. Every group is listed in [maintainer-groups.yaml](../maintainer-groups.yaml).

Membership in a group uses the following:

- There will be at least 3 people
- Any Helm maintainer is eligible
- A Helm maintainer joins a group by a simple majority vote of the Helm maintainers
- A member may step down by emailing the [Helm mailing list](https://lists.cncf.io/g/cncf-helm).
- Members MUST remain active Helm maintainers. A member who becomes emeritus, or who is unresponsive for > 3 months, leaves the group unless a [super-majority](https://en.wikipedia.org/wiki/Supermajority#Two-thirds_vote) of the other members agrees to extend the period to be greater than 3 months

Once voted in, a member stays in a group until stepping down (or, in rare cases, being removed).

#### GitHub Admin

The GitHub Admin group holds the owner role on the [Helm GitHub organization](https://github.com/helm). Keeping that role with a few people limits the damage a compromised account or a mistake can do.

Members of the group are responsible for:

- Creating, transferring, archiving, and deleting repositories once the maintainers or the project involved has decided to
- Adding and removing organization members and teams so that GitHub access matches each project's record of maintainers
- Managing organization settings, secrets, and security policies (e.g., requiring two-factor authentication)
- Approving and removing GitHub Apps, bots, and other integrations
- Controlling access to other Helm assets where an administrator is needed, such as hosting and project calendars

The group acts on decisions made elsewhere. It has no decision making power of its own beyond what is needed to keep the organization secure, and in an emergency (e.g., a compromised account) a member may act first and report to the maintainers afterwards. Requests for the group are made by opening an issue in the [community repository](https://github.com/helm/community) or by emailing the group.

The group started with the people who were Helm org maintainers when this structure was adopted. Its members are listed in [MAINTAINERS.md](../MAINTAINERS.md).

#### Security Team

The security team responds to vulnerability reports. The team's responsibilities and process are described in [SECURITY.md](../SECURITY.md).

## Decision Making at the Helm org level

When maintainers need to make decisions there are two ways decisions are made, unless described elsewhere.

The default decision making process is [lazy-consensus](http://communitymgt.wikia.com/wiki/Lazy_consensus). This means that any decision is considered supported by the team making it as long as no one objects. Silence on any consensus decision is implicit agreement and equivalent to explicit agreement. Explicit agreement may be stated at will.

When a consensus cannot be found a maintainer can call for a [majority](https://en.wikipedia.org/wiki/Majority) vote on a decision.

Many of the day-to-day project maintenance can be done by a lazy consensus model. But the following items must be called to a vote of the Helm maintainers:

- Enforcing a CoC violation (super majority)
- Removing a maintainer for any reason other than inactivity (super majority)
- Changing the governance rules (this document) (super majority)
- Licensing and intellectual property changes (including new logos, wordmarks) (simple majority)
- Adding, archiving, or removing projects (simple majority)
- Creating or dissolving groups (simple majority)
- Adding a maintainer to a group (simple majority)
- Spending Helm/CNCF money on anything CNCF deems "not cheap and easy" (simple majority)

Every Helm maintainer has one vote. A vote is open for a minimum of three business days and stays open until a majority of the Helm maintainers has voted. The result is counted from the votes cast.

Other decisions may, but do not need to be, called out and put up for decision on the [Helm mailing list](https://lists.cncf.io/g/cncf-helm) at any time and by anyone. By default, any decisions called to a vote will be for a _simple majority_ vote.

## Code of Conduct

The code of conduct is overseen by the Helm maintainers. Possible code of conduct violations should be emailed to the maintainers at <cncf-helm-maintainers@lists.cncf.io>.

If the possible violation is against one of the maintainers that member will be recused from voting on the issue. Such issues must be escalated to the appropriate CNCF contact, and CNCF may choose to intervene.

## DCO and Licenses

The following licenses and contributor agreements will be used for Helm projects:

- [Apache 2.0](https://opensource.org/licenses/Apache-2.0) for code
- [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode) for documentation
- [Developer Certificate of Origin](https://developercertificate.org/) for new contributions
