# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Repository Purpose

This is the **Helm community** repository — it contains governance documents, Helm Improvement Proposals (HIPs), security policies, and project operations documentation for the Helm project generally.
It is **not** the Helm source code (do not suggest source code changes here). Content from this repo is imported into the Helm website for end-user consumption (helm.sh) via the `helm/helm-www` repository's `download-remote-community` command.

There is no build system, test suite, or CI pipeline for this repo. All content is Markdown and YAML.

## Contributing

All commits must be signed off (DCO). Content is licensed under Creative Commons Attribution 4.0. Each PR needs 2 approvals from maintainers (org maintainers for process HIPs, project maintainers for feature/informational HIPs).

## HIPs (Helm Improvement Proposals)

HIPs live in `hips/hip-NNNN.md`. The canonical format is defined in `hips/hip-0001.md`.

Use the `hip-author` skill (`.agents/skills/hip-author/`) for guided HIP creation — it walks through idea refinement, scope narrowing, and section-by-section drafting with the correct template and length targets.

### Quick reference

- Preamble: YAML frontmatter with `hip`, `title`, `authors`, `created`, `type`, `status` (see `hip-0001.md` for full schema)
- Placeholder number: always use `hip: 9999` on initial submission
- Status: always `draft` on initial submission
- Types: `feature`, `informational`, `process`
- Required sections: Abstract, Motivation, Rationale, Specification, Backwards compatibility, Security implications, How to teach this, Reference implementation, Rejected ideas, Open issues, References
- Auxiliary files: `proposal-XXX-YY.ext` naming convention
- Update `hips/README.md` when adding a new HIP

## Key Files

- `governance/governance.md` — full governance rules (maintainer structure, voting, decision-making)
- `SECURITY.md` — vulnerability reporting process and security team membership
- `maintainer-groups.yaml` — structured data of all maintainer groups, their repos, and mailing lists
- `MAINTAINERS.md` — current Helm org maintainers

## Helm source

The main Helm source code lives in the [helm/helm](https://github.com/helm/helm) Github repository. Along-side additional Helm project repositories in the Helm github org: <https://github.com/helm>.
