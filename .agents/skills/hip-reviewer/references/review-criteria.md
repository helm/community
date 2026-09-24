# HIP Review Criteria Reference

This file provides the detailed evaluation criteria for reviewing Helm Improvement Proposals. The skill's SKILL.md defines the review process; this file provides the domain knowledge needed to apply each criterion.

## Helm's mission and scope

Helm is the package manager for Kubernetes. Its core functions are:

- **Create**: package Kubernetes resources into charts
- **Distribute**: share charts via repositories (HTTP or OCI registries)
- **Manage**: install, upgrade, rollback, and uninstall releases on clusters

Proposals that align with these functions have a natural home in Helm. Proposals that push Helm toward being an application platform, a CI/CD tool, a service mesh manager, or a cluster provisioner are likely out of scope — even if they touch Kubernetes.

## Backwards compatibility rules (HIP-0004)

Changes to Helm minor and patch releases must be 100% backwards compatible per Semantic Versioning. The following rules apply:

### CLI
- Commands and flags must not be removed, renamed, or moved
- Commands and flags cannot be repurposed to provide different behavior
- Flag types must not change (unless the new type is a superset: int8 → int32, int → float, float → string)
- Short name and long name rules are independent — changing one while keeping the other is still a break

### CLI output
- Help text may change
- Structured output format (tables, JSON, YAML) must not change
- Error messages can change if doing so increases usability
- Bug fixes (spelling, misinformation) are allowed
- Return codes should not change unless incorrect

### File formats (Chart.yaml, index.yaml, repositories.yaml, values.yaml)
- No fields may be removed from any of these files
- No fields may be added or modified on Chart.yaml
- Added fields in other files must be optional and accompanied by backward compatibility checks
- No existing optional field can be made required
- No 3rd-party-specific fields (use annotations instead)

### Charts
- Template handling must not change
- Reserved directory names (e.g., crds) must not be removed
- Allowed file types cannot become disallowed
- New objects or file types added to charts must be optional

### Templates (functions, syntax, variables)
- Template functions cannot be removed
- Return types cannot change
- Function signatures can only grow additively (new optional parameters)
- Built-in directives, constants, and variables cannot be removed or changed in breaking ways
- Template syntax cannot change (except upstream Go template changes)

### Exceptions
- Backwards compatibility may be broken for security reasons
- Experimental features behind feature flags are exempt from these rules (but cannot break non-experimental features)

## User base breadth

Helm serves a wide range of users:

- **Chart authors** writing and maintaining charts
- **Chart consumers** installing and managing releases
- **Platform/ops teams** managing Helm across fleets of clusters
- **Tool builders** integrating Helm's SDK or CLI into CI/CD pipelines, GitOps tools, etc.
- **Registry operators** hosting chart repositories

A change that serves only one narrow segment (e.g., only benefits users of a specific cloud provider, or only helps teams using a particular deployment pattern) needs to justify why it belongs in Helm core rather than in a plugin, chart convention, or external tool.

Questions to ask:
- What fraction of Helm users will benefit from this change?
- Does this add complexity for users who don't need the feature?
- Could this be implemented as a plugin instead of a core feature?
- Does the feature require all users to learn new concepts even if they don't use it?

## Implementation complexity signals

Helm is maintained by a small group of volunteers. Implementation burden is a real factor in whether a HIP can succeed:

**Red flags:**
- Requires changes across multiple Helm projects (helm, helm-www, chartmuseum, etc.)
- Introduces new long-lived infrastructure the maintainers must operate
- Requires ongoing maintenance of external integrations
- Adds significant new surface area to the Go SDK that must be kept stable
- Depends on features that don't exist yet in Kubernetes or Go
- Requires coordination with external projects for correctness

**Green flags:**
- Self-contained change within a single package
- Clear, bounded scope with a definitive "done" state
- Author has a working proof-of-concept
- Follows established patterns in the codebase

## Security evaluation checklist

When reviewing security implications, consider:

1. **Data exposure**: Does this feature expose data that wasn't previously accessible? Could chart values, release secrets, or cluster state leak?
2. **Trust boundaries**: Does this change who can see or modify what? Does it add new trust relationships?
3. **Supply chain**: Does this affect chart provenance, signing, or verification? Could a malicious chart exploit this feature?
4. **Privilege escalation**: Could this be used to gain Kubernetes permissions beyond what the user already has?
5. **Denial of service**: Could this be used to exhaust cluster resources, fill storage, or create runaway processes?
6. **Default safety**: Are the defaults secure? Does the user have to opt-in to risky behavior, or opt-out?

A well-written HIP addresses these proactively. A poorly-written one says "No security implications" for a feature that clearly touches trust boundaries.
