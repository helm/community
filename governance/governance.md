# Helm Governance

The following document outlines how the Helm project governance operates.

## The Helm Project

The Helm project is made up of several codebases and services with different release cycles that enable users to create, distribute, and operate packages in Kubernetes. These codebases include:

* Helm - the package manager
* Charts - community curated charts
* Monocular - a web interface for browsing one or more chart repositories
* Chartmuseum - a chart repository with support for pushing charts, auth, and more
* chart-testing - container based chart testing tools

The services provided include:

* Documentation for those who want to create, distribute, depend on, and operate packages
* Public chart search and discovery

## Maintainers Structure

There are two levels of maintainers for Helm. The Helm org maintainers oversee the overall project and it's health. Project maintainers focus on a single codebase or a group of related codebases. For example, the chart maintainers manage the charts repository and a few repositories that support charts. The Helm org maintainers are responsible for the overall health of the project while the project maintainers are responsible for maintaining the code.

Changes in maintainership have to be announced on the [helm mailing list](https://lists.cncf.io/g/cncf-helm).

### Helm Org Maintainers

The Helm Org maintainers are responsible for:

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

Changes to org maintainers use the following:

* There will be between 3 and 9 people (maintaining an odd number). If there is a loss of a org maintainer bring the count to an even number the org maintainers can continue with an even number as long as they are actively going through the process to bring the maintainer count back to an odd number
  * To ensure diverse representation from all areas of the Helm org, the breakdown of Helm Org Maintainers group will be as follows:
    * When there are 3 Org Maintainers
      * 1 Representative from the Helm core project
        * Also serves as the default chair unless someone else in the Org Maintainers group is interested in the role
      * 1 Representative from the Charts project
      * 1 Representative from another Helm project
    * When there are 5 Org Maintainers
      * 1 Chair (from any project under the Helm org)
      * 2 Representatives from the Helm core project
      * 1 representative from the Charts project
    * 1 representative from another Helm project
    * When there are 7 Org Maintainers
      * 1 Chair (from any project under the Helm org)
      * 3 Representatives from the Helm core project
      * 1 Representative from the Charts project
      * 2 Representatives from another Helm project
    * When there are 9 Org Maintainers
      * 1 Chair (from any project under the Helm org)
      * 4 Representatives from the Helm core project
      * 2 Representatives from the Charts project
      * 2 Representatives from another Helm project
* Any project maintainer is eligible for a position as an org maintainer
* No one company or organization can employ a simple majority of the org maintainers
* An org maintainer may step down by emailing the org maintainers mailing list. Within 7 calendar days the [helm mailing list](https://lists.cncf.io/g/cncf-helm) needs to be notified of the change
* Org maintainers MUST remain active on the project. If they are unresponsive for > 3 months they will lose org maintainership unless a [super-majority](https://en.wikipedia.org/wiki/Supermajority#Two-thirds_vote) of the other org maintainers agrees to extend the period to be greater than 3 months
* When there is an opening for a new org maintainer, any person who has made a contribution to any repo under the Helm GitHub org may nominate a suitable project maintainer as a replacement
  * The nomination period will be three weeks starting the day after an org maintainer opening becomes available
  * The nomination must be made via the [public Helm mailing list](https://lists.cncf.io/g/cncf-helm/)
* When nominated individual(s) agrees to be a candidate for maintainership, the project maintainers may vote
  * The voting period will be open for a minimum of three business days and will remain open until a super-majority of project maintainers has voted
  * Only current project maintainers are eligible to vote
  * When the number of nominated individuals matches the number of openings each individual needs to have a yes vote from a super-majority of those that voted
  * When there are more individuals than open positions voting will use [Condorcet](https://en.wikipedia.org/wiki/Condorcet_method) ranking on [CIVS](http://civs.cs.cornell.edu/) using the [Schulze method](https://en.wikipedia.org/wiki/Schulze_method)
* When an org maintainer steps down, they become an emeritus maintainer

Once an org maintainer is elected, they remain a maintainer until stepping down (or, in rare cases, are removed). Voting for new maintainers occurs when necessary to fill vacancies. Any existing project maintainer is eligible to become an org maintainer.

### Project Maintainers

Project maintainers are responsible for activities surrounding the development and release of code and the operation of any services they own (e.g., the documentation site). Technical decisions for code resides with the project maintainers unless there is a decision related to cross maintainer groups that cannot be resolved by those groups. Those cases can be esclated to the project maintainers.

In some cases a groups of maintainers are responsible for more than one repo (e.g., charts maintainers managing the charts, chart-testing, charts-tooling). In other cases the maintainers are responsible for a single project (e.g., chartmuseum or monocular).

Project maintainers do not need to be software developers. No explicit role is placed upon them and they can be anyone appropriate for the work being produced. For example, if a repository is for documentation it would be appropriate for maintainers to be editors.

Changes to maintainers use the following:

* A maintainer may step down by emailing the [helm mailing list](https://lists.cncf.io/g/cncf-helm)
* Maintainers MUST remain active. If they are unresponsive for > 3 months they will be automatically removed unless a [super-majority](https://en.wikipedia.org/wiki/Supermajority#Two-thirds_vote) of the other project maintainers agrees to extend the period to be greater than 3 months
* New maintainers can be added to a project by a [super-majority](https://en.wikipedia.org/wiki/Supermajority#Two-thirds_vote) vote of the existing maintainers
* When a project has no maintainers the helm org maintainers become responsible for it and may archive the project or find new maintainers

## Decision Making at the Helm org level

When maintainers need to make a decisions there are two ways decisions are made, unless described elsewhere.

The default decision making process is [lazy-consensus](http://communitymgt.wikia.com/wiki/Lazy_consensus). This means that any decision is considered supported by the team making it as long as no one objects. Silence on any consensus decision is implicit agreement and equivalent to explicit agreement. Explicit agreement may be stated at will.

When a consensus cannot be found a maintainer can call for a [majority](https://en.wikipedia.org/wiki/Majority) vote on a decision.

Many of the day-to-day project maintenance can be done by a lazy consensus model. But the following items must be called to vote:

* Enforcing a CoC violation (super majority)
* Removing a maintainer for any reason other than inactivity (super majority)
* Changing the governance rules (this document) (super majority)
* Licensing and intellectual property changes (including new logos, wordmarks) (simple majority)
* Adding, archiving, or removing subprojects (simple majority)
* Utilizing Helm/CNCF money for anything CNCF deems "not cheap and easy" (simple majority)

Other decisions may, but do not need to be, called out and put up for decision on the [helm mailing list](https://lists.cncf.io/g/cncf-helm) at any time and by anyone. By default, any decisions called to a vote will be for a _simple majority_ vote.

## Code of Conduct

The code of conduct is overseen by the Helm project maintainers. Possible code of conduct violations should be emailed to the project maintainers at cncf-helm-maintainers@lists.cncf.io.

If the possible violation is against one of the project maintainers that member will be recused from voting on the issue. Such issues must be escalated to the appropriate CNCF contact, and CNCF may choose to intervene.

## DCO and Licenses

The following licenses and contributor agreements will be used for Helm projects:

* [Apache 2.0](https://opensource.org/licenses/Apache-2.0) for code
* [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode) for documentation
* [Developer Certificate of Origin ](https://developercertificate.org/) for new contributions