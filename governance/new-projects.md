# Adopting New Projects

Helm is an organization with multiple projects. This document describes how new projects may be added to the Helm organization.

## Legal Requirements

Any project added must meet the CNCF's legal requirements, including licenses (Apache 2) and [intellectual property commitments](https://github.com/cncf/foundation/blob/master/copyright-notices.md).

## Governance Requirements

Any project added must (a) agree to the Helm organization governance, and (b) have in a place a governance structure that meets the requirements set out by the Helm organization in [our governance documentation](governance.md).

Additionally, governance requirements from CNCF must be adhered to. For example, any candidate project must use the CNCF's [Code of Conduct](https://github.com/cncf/foundation/blob/master/code-of-conduct.md).

We currently do not allow any projects to "bring their own" code of conduct or to make any additional changes to the existing CNCF code of conduct.

## Security Requirements

Any project added must have security guidelines for how to report and resolve security issues. It is acceptable to merely use the existing Helm process for this. But projects that require special or additional reporting may do so.

## Process

### 1. Project opens issue

The candidate project must open an issue in the [Helm community repository](http://github.com/helm/community) requesting that it join.

This issue should provide:

- A link back to the project
- The GitHub user names of those in a governance role for that project
- A short description (1-3 paragraphs) of why it makes sense to add to Helm
- A short explanation of how the project is to be staffed going forward.

### 2. Helm Org Review

The Helm organization owners will review the proposal. This process will likely involve some back-and-forth conversation with the owners of the project.

### 3. Helm Org Voting

As stated in the [Helm governance](governance.md), a candidate project must pass a supermajority vote before it is accepted into the Helm organization.

If the Helm org maintainers vote to include the project, the project will be considered a _recommended Helm project_, and CNCF will be notified of our intentions.

### 3. CNCF Review

CNCF will apply its reviews, including but not limited to intellectual property reviews. This process will be managed between designated Helm org maintainers and the CNCF, with the project being included as necessary.

If this review fails, projects will be given the opportunity to comply with CNCF's requirements. Failure to comply, though, will remove the project from _recommended project_ status, and the process will discontinue.

### 4. Transition of Resources

Once both Helm and CNCF have reviewed the project and approved, the project becomes a _transitional project_. Helm org maintainers will work with the transitional project to move resources, transfer requisite ownership, and make any other necessary changes.

### 5. Completion

At the completion of the transitional process, the project will be an official _Helm project_. At the next governance cycle, it will be granted representation as per [the governance documents](governance.md).