# HIP Template and Reference

This file contains the canonical HIP template extracted from hip-0001.md, enriched with guidance and length recommendations derived from analyzing all existing HIPs in the repository.

## Preamble format

Every HIP starts with YAML frontmatter. Headers marked with `*` are optional.

```yaml
---
hip: 9999
title: ""
authors: [ "Full Name <email@example.com>" ]
created: "YYYY-MM-DD"
type: "feature"
status: "draft"
---
```

### Preamble fields

| Field | Required | Notes |
|-------|----------|-------|
| `hip` | yes | Use `9999` as placeholder on initial submission (real number assigned at merge) |
| `title` | yes | Short descriptive title |
| `authors` | yes | Format: `"Name <email>"` or `"Name"` (email optional) |
| `created` | yes | RFC 3339 date (YYYY-MM-DD) |
| `type` | yes | `feature`, `informational`, or `process` |
| `status` | yes | Always `draft` on initial submission |
| `helm-version`* | no | Helm version this ships with (add after implementation ships) |
| `requires`* | no | List of HIP numbers this depends on |
| `replaces`* | no | HIP number this obsoletes |
| `superseded-by`* | no | HIP number that supersedes this one |

### Type definitions

- **feature**: proposes a new feature or implementation for a Helm project, or an interoperability standard
- **informational**: design guidance or information for the community; non-binding
- **process**: changes to project processes, governance, or tooling; requires Helm maintainer approval

## Required sections with length guidance

Length recommendations are based on analysis of all 26 existing HIPs. The "typical" range covers the 25th-75th percentile. These are guidelines, not rules — a complex feature HIP will naturally be longer than a simple process HIP.

Aim for the shortest version that fully meets each section's purpose. The **target** marks what is usually *enough*; the **typical** range is a ceiling to stay within, not a quota to reach. Prefer focus over length — padding a section to hit a number buries its substance.

### Abstract
**Typical: 25-100 words. Target: ~50 words. Max observed: 185 words.**

A concise, self-contained description of what the proposal does and what problem it solves. A reader should understand the proposal's purpose without reading further. Avoid implementation details — save those for the Specification.

Most accepted HIPs have abstracts well under 200 words. The hip-0001 guidance says "~200 words" but in practice shorter is better.

### Motivation
**Typical: 100-200 words. Target: ~150 words. Max observed: 488 words.**

This is the most critical section for acceptance. Proposals without sufficient motivation are rejected outright. Structure around:

- **Current limitations** — what can't users do today, or what is painful?
- **Real-world impact** — concrete examples, not hypotheticals. Name the roles affected (chart authors, cluster operators, platform teams, CI/CD pipelines).
- **Existing workarounds** — what do people do today, and why is it inadequate?

Subheadings work well here (e.g., "### Current Limitations", "### Who Is Affected", "### Real-World Impact"). Most strong HIPs use them.

### Rationale
**Typical: 50-150 words. Target: ~100 words. Max observed: 4632 words (outlier).**

Why this design, not just what. Cover:

- Why this approach over alternatives
- How similar problems are handled in other projects (if relevant)
- Key tradeoffs and why they're acceptable
- Naming decisions (if the proposal introduces new concepts or API surface)

Use subheadings to separate distinct design decisions (e.g., "### Why X over Y", "### Key Tradeoffs").

### Specification
**Typical: 230-680 words. Target: ~350 words. Max observed: 1563 words.**

The largest section in most HIPs. For feature HIPs, this must be detailed enough for someone else to implement. Include:

- Data structures, Go types, CLI flags, API shapes
- Usage examples (CLI invocations, YAML snippets, template examples)
- Behavior matrix if there are multiple modes or contexts (see hip-0029 for a good example)
- Edge cases and how they're handled

For process HIPs, define concrete steps, timelines, roles, and decision criteria rather than code.

### Backwards compatibility
**Typical: 15-90 words. Target: ~30 words. Max observed: 177 words.**

Often short. For purely additive changes, a sentence confirming no breakage is sufficient. For breaking changes, be explicit about:

- What breaks and severity
- Migration path
- Whether this requires a major version bump (Helm follows SemVer)

### Security implications
**Typical: 1-40 words. Target: ~20 words. Max observed: 532 words.**

Often very short for benign features ("No security implications"). For features that touch data exposure, authentication, or trust boundaries, cover:

- Attack surface added
- Data exposure risks
- How a malicious chart or user could abuse the feature
- Mitigations built into the design

### How to teach this
**Typical: 25-70 words. Target: ~50 words. Max observed: 569 words.**

What documentation needs to change and what key patterns users should learn:

- Documentation additions or modifications needed
- The one example you'd lead with when explaining the feature
- How it fits into concepts users already know

### Reference implementation
**Typical: 5-20 words. Target: ~15 words. Max observed: 133 words.**

Usually very short. Link to a PR or describe what the implementation will involve. The implementation must be complete before the HIP can reach "final" status, but it does not need to exist when the HIP is submitted as a draft.

### Rejected ideas
**Typical: 10-100 words. Target: ~50 words. Max observed: 1171 words.**

Ideas discussed and discarded, with brief explanations of why. Prevents reviewers from re-proposing the same alternatives. Use a bulleted list for multiple rejected ideas — each entry should be 1-2 sentences.

### Open issues
**Typical: 1-50 words. Target: ~15 words. Max observed: 177 words.**

Unresolved questions. Fine to submit a draft HIP with open issues — they're resolved during review. Can also be empty if all questions are settled.

### References
**Typical: 10-40 words. Target: ~20 words. Max observed: 420 words.**

URLs, specs, and materials referenced throughout the HIP. Use markdown link format.

## Optional sections

These sections appear in some existing HIPs and may be useful depending on the proposal:

- **Scope** (hip-0026) — when a feature HIP is large, explicitly defining what is and isn't in scope helps focus review
- **Implementation Plan** (hip-0026) — for complex features, a phased delivery plan with milestones
- **Prior raised issues** (hip-0025) — links to related GitHub issues or mailing list threads that preceded the HIP

## Auxiliary files

Diagrams or supporting files use the naming convention: `proposal-XXX-YY.ext` (e.g., `proposal-029-01.png`), where XXX is the HIP number and YY is a serial number starting at 01.

## Approval requirements

- Feature and informational HIPs: at least 2 approvals from project maintainers
- Process HIPs: at least 2 approvals from Helm maintainers

## Status lifecycle

`draft` -> `accepted` -> `final` (after reference implementation is complete)

Other statuses: `provisional` (accepted but needs more feedback), `deferred` (paused), `rejected`, `superseded`
