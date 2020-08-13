---
proposal: 0001
title: "Writing a Proposal"
authors: [ "Matthew Fisher <matt.fisher@microsoft.com" ]
created: 2020-08-13
---

A proposal is a design document providing information to the Helm community, or
describing a new feature for a Helm project or its processes or environment. The
proposal should provide a concise technical specification of the feature and a
rationale for the feature.

We intend proposals to be the primary mechanisms for proposing major new
features, for collecting community input on an issue, and for documenting the
design decisions that have gone into the project. The proposal author is
responsible for building consensus within the community and documenting
dissenting opinions.

The system is heavily influenced by Python's Enhancement Proposal (PEP) system,
hence the name. Given that Helm is a much (MUCH) smaller project than Python,
there are a few changes in the proposal workflow to make it a looser format for
proposals, most notably the lack of proposal "editors", a BDFL and a much
smaller review & resolution cycle for proposals.

Because the proposals are maintained as markdown files in a versioned
repository, their revision history is the historical record of the proposal.

## Proposal workflow

### Start with an idea for Helm

The proposal process begins with a new idea for the Helm project. It is highly
recommended that a single proposal contain a single key proposal or new idea.
Small enhancements or patches often don't need a proposal and can be injected
into a project's development workflow with a patch submission to the project's
issue tracker. The more focused the proposal, the more successful it tends to
be. As per Helm's governance, the project maintainers reserve the right to
reject proposal proposals if they appear too unfocused, too broad, or
incomplete. If in doubt, split your proposal into several well-focused ones, or
explain your proposal more clearly.

Each proposal must have a champion -- someone who writes the proposal using the
style and format described below, shepherds the discussions in the appropriate
communication channels, and attempts to build community consensus around the
idea. The proposal champion (a.k.a. author) should first attempt to ascertain
whether the idea is proposal-able. Posting to the [cncf-helm][] mailing list is
the best way to go about this.

Vetting an idea publicly before going as far as writing a proposal is meant to
save the potential author time. Many ideas have been brought forward for
changing Helm that have been rejected for various reasons. Asking the Helm
community first if an idea is original helps prevent wasting time on something
that is guaranteed to be rejected based on prior discussions (searching the
internet does not always do the trick). It also helps to make sure the idea is
**applicable to the entire community and not just the author**. Just because an
idea sounds good to the author does not mean it will work for most Helm users in
most areas where Helm is used.

Once the champion has asked the Helm community as to whether an idea has any
chance of acceptance, a summary of the proposal should be presented to
[cncf-helm][]. This gives the author a chance to flesh out the proposal as a
high quality, well-formatted proposal, and to address initial concerns about the
proposal.

### Submitting a proposal

Following a discussion on [cncf-helm][], the proposal should be submitted as a
proposal via a [GitHub pull request][1]. The proposal must be written in
proposal style as described below, else it will fail review immediately
(although minor errors may be corrected by the maintainers).

The standard proposal workflow is:

- You, as the proposal author, fork the helm/community repository and create a
  new proposal
- Proposals are written in [Markdown][markdown]
- Push your proposal to your GitHub fork and [submit a pull request][pr].

Once the review process is complete, the project maintainers will approve the
proposal by commenting on the proposal with a "Looks good to me", or "LGTM" for
short. (note that this is not the same as accepting your proposal!)

The project maintainers will not unreasonably deny a proposal. Reasons for
denying proposal status include duplication of effort, being technically
unsound, not providing proper motivation, not addressing backwards
compatibility, or not in keeping with Helm's design. The core maintainers can be
consulted during the approval phase.

Once approved, proposal authors are responsible for collecting community
feedback on a proposal before submitting a feature that satisfies the proposal
for review. However, wherever possible, long open-ended discussions on public
mailing lists should be avoided. Strategies to keep the discussions efficient
include: setting up a separate meetings for the topic, having the proposal
author accept private comments in the early design phases, setting up a wiki
page, etc. proposal authors should use their discretion here.

## What belongs in a successful proposal

Each proposal should have the following parts:

1. Preamble - YAML style headers containing metadata about the proposal,
   including the proposal number, a short descriptive title, the author names,
   etc.
1. Abstract - a short (~200 word) description of the technical issue being
   addressed.
1. Specification - The technical specification should describe the syntax and
   semantics of any new feature. If the proposal describes a standard, the
   specification should be detailed enough to describe how other community
   members may re-implement the standard in their product.
1. Motivation - The motivation is critical for proposals that want to change
   Helm in a significant way. It should clearly explain why existing features
   are inadequate to address the problem that the proposal solves. proposal
   submissions without sufficient motivation may be rejected outright.
1. Rationale - The rationale fleshes out the specification by describing what
   motivated the design and why particular design decisions were made. It should
   describe alternate designs that were considered and related work, e.g. how
   the feature is supported in other projects.
1. Backwards Compatibility - All proposals that introduce backwards
   incompatibilities must include a section describing these incompatibilities
   and their severity. The proposal must explain how the author proposes to deal
   with these incompatibilities. proposal submissions without a sufficient
   backwards compatibility treatise may be rejected outright.
1. Reference Implementation - The reference implementation must be completed
   before any proposal is given status "Final", but it need not be completed
   before the proposal is accepted. While there is merit to the approach of
   reaching consensus on the specification and rationale before writing code,
   the principle of "rough consensus and running code" is still useful when it
   comes to resolving many discussions of API details.

The final implementation must include test code and documentation appropriate
for the Helm project's reference or the community at large.

## Proposal header preamble

Each proposal must begin with a [YAML][yaml] style header preamble. The headers
must appear in the following order:

Headers marked with "*" are optional and are described below. All other headers
are required.

```yaml
---
proposal: <proposal number>
title: "<proposal title>"
authors: [ "<list of authors' real names and optionally, email addresses>" ]
created: 2020-08-13
* helm-version: "<version number>"
* requires: [ <proposal numbers> ]
* replaces: <proposal number>
* superseded-by: <proposal number>
---
```

The `authors` header lists the names, and optionally the email addresses of all
the authors/owners of the proposal. The format of the author header value must
be

```yaml
authors: [ "John Doe <john@example.com>", "Jane Doe <jane@example.com>" ]
```

If the email address is included, and

```yaml
authors: [ "John Doe", "Jane Doe" ]
```

If the address is not given.

The `created` header records the date that the proposal was published to the
Helm project on Github. The header should be in [RFC 3339][rfc3339] format, e.g.
2020-08-13.

Proposals will typically have a `helm-version` header which indicates the
version of Helm that the feature will be released with. proposals without a
`helm-version` header indicate interoperability standards that will initially be
supported through external libraries and tools, and then supplemented by a later
proposal to add support to the project's standard library or Command Line
Interface. Informational and process proposals do not need a `helm-version`
header.

Proposals may have a `requires` header, indicating the proposal numbers that
this proposal ends on.

Proposals may also have a `superseded-by` header indicating that a proposal has
been rendered obsolete by a later document; the value is the number of the
proposal that replaces the current document. The newer proposal must have a
`replaces` header containing the number of the proposal that it rendered
obsolete.

## Auxiliary files

proposals may include auxiliary files such as diagrams. Such files must be named
proposal-XXX-YY.ext, where "XXX" is the proposal number, "YY" is a serial number
(starting at 01), and "ext" is replaced by the actual file extension (e.g.
"png").

## Transferring proposal ownership

It occasionally becomes necessary to transfer ownership of proposals to a new
champion. In general, it is preferable to retain the original author as a
co-author of the transferred proposal, but that's really up to the original
author. A good reason to transfer ownership is because the original author no
longer has the time or interest in updating it or following through with the
proposal process, or has fallen off the face of the earth (i.e. is unreachable
or not responding to email). A bad reason to transfer ownership is because the
author doesn't agree with the direction of the proposal. One aim of the proposal
process is to try to build consensus around a proposal, but if that's not
possible, an author can always submit a competing proposal.

If you are interested in assuming ownership of a proposal, you can also do this
via pull request. Fork the helm/community repository, make your ownership
modification, and submit a pull request. You should also send a message asking
to take over, addressed to both the original author and the org maintainers
(`@helm/org-maintainers`). If the original author doesn't respond to email in a
timely manner, the org maintainers will make a unilateral decision (it's not
like these decisions can't be reversed).

[cncf-helm]: mailto:cncf-helm@lists.cncf.io
[markdown]: https://daringfireball.net/projects/markdown/syntax
[pr]: https://github.com/helm/community/pulls
[rfc3339]: https://tools.ietf.org/html/rfc3339
[yaml]: https://yaml.org/
