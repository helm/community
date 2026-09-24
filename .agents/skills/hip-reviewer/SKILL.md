---
name: hip-reviewer
description: "Review and critique Helm Improvement Proposals (HIPs) from a maintainer perspective. Use this skill when asked to review a HIP, evaluate a proposal, provide feedback on a HIP, critique a HIP, or assess whether a proposed change to Helm is appropriate. Also trigger when the user says things like 'what do you think of this HIP', 'review this proposal', 'is this HIP ready', 'give feedback on hip-NNNN', or pastes HIP content for evaluation."
---

# HIP Review Guide

This skill helps Helm maintainers and community members evaluate proposed HIPs against the project's standards and values. The review produces a structured assessment covering five dimensions, with a clear verdict and actionable feedback.

Read `references/review-criteria.md` for the detailed domain knowledge (backwards compatibility rules from HIP-0004, Helm's mission scope, user base breadth expectations, implementation complexity signals, and security checklist).

## Review Process

### Step 1: Read the HIP

If the user provides a HIP number (e.g., "review hip-0029"), read it from `hips/hip-NNNN.md`. If they paste content directly, work from that.

### Step 2: Evaluate against five dimensions

Assess the HIP on each dimension below. For each, provide:
- A **rating**: Strong / Adequate / Weak / Insufficient
- A brief explanation (2-4 sentences) justifying the rating
- Specific concerns or suggestions if the rating is below "Strong"

#### 1. Motivation and Rationale

Does this proposal solve a real problem for Helm users? Is the motivation grounded in concrete pain points, or is it speculative? Does it align with Helm's core mission (package, distribute, and manage Kubernetes applications)?

**Strong**: Clear user pain, concrete examples, obvious alignment with Helm's purpose.
**Weak**: Vague motivation, hypothetical benefits, or solves a problem that isn't really Helm's to solve.

Look for: named user roles affected, existing workarounds described, real-world scenarios (not just "it would be nice if...").

#### 2. Change Applicability (Breadth)

Will this benefit a broad cross-section of Helm users, or is it niche? Does it introduce complexity that all users must contend with even if they don't use the feature?

**Strong**: Benefits most Helm users or a large segment; complexity is opt-in and invisible to non-users.
**Weak**: Benefits only a narrow use case; adds concepts or flags everyone encounters; could be a plugin instead.

Ask yourself: would a typical chart author and a typical cluster operator both understand why this exists? If neither would notice it in their daily workflow, it may be too niche for core.

#### 3. Backwards Compatibility

Does the proposal respect HIP-0004's compatibility rules? If it introduces breaking changes, does it properly scope them to a major version or use experimental feature flags?

**Strong**: Purely additive, no breakage, existing users/charts/tools unaffected.
**Adequate**: Minor changes with clear migration path, properly acknowledged.
**Weak/Insufficient**: Breaks CLI contracts, file formats, template behavior, or Go API without acknowledging it or providing mitigation.

Check the specific rules in `references/review-criteria.md` — CLI flags, file format fields, template functions, return types, structured output. Authors often miss subtle breaks like changing the type of a field or adding a required parameter.

#### 4. Implementation Complexity and Feasibility

Is this implementable by the Helm maintainer team (small volunteer group)? Is the scope bounded? Does a proof-of-concept exist or is it purely theoretical?

**Strong**: Bounded scope, single-project change, proof-of-concept exists, follows existing patterns.
**Weak**: Requires multi-project coordination, unbounded ongoing maintenance, depends on unmerged upstream features, or no one has demonstrated feasibility.

Consider: who will actually implement this? If the author isn't planning to, is there a realistic path to someone else doing it? Orphaned HIPs with no implementor become dead weight.

#### 5. Security Implications

Has the proposal thoroughly considered security? Are the defaults safe? Does it add attack surface, expose data, or change trust boundaries?

**Strong**: Proactively addresses security, defaults to safe behavior, minimal new attack surface.
**Weak**: Dismisses security ("No security implications") for a feature that clearly touches trust or data boundaries. Missing analysis of how a malicious chart or user could abuse the feature.

Use the checklist in `references/review-criteria.md`: data exposure, trust boundaries, supply chain, privilege escalation, denial of service, default safety.

### Step 3: Produce the review

Structure the output as follows:

```
## HIP Review: [HIP title]

### Summary Verdict

[One sentence: accept / request changes / major concerns]

### Dimension Ratings

| Dimension | Rating | Key Concern |
|-----------|--------|-------------|
| Motivation & Rationale | [rating] | [one-line summary] |
| Change Applicability | [rating] | [one-line summary] |
| Backwards Compatibility | [rating] | [one-line summary] |
| Implementation Feasibility | [rating] | [one-line summary] |
| Security Implications | [rating] | [one-line summary] |

### Detailed Assessment

[2-4 sentences per dimension, with specific line references or quotes from the HIP where relevant]

### Recommendations

[Bulleted list of specific, actionable changes the author should make]
```

### Verdict guidance

- **All Strong/Adequate**: "This HIP is well-prepared for acceptance. Minor suggestions below."
- **One or more Weak**: "This HIP needs revisions before it's ready for acceptance." Focus recommendations on the weak areas.
- **Any Insufficient**: "This HIP has fundamental issues that need to be addressed before further review." Explain what's missing.

## Tone

Be constructive but honest. The goal is to help the author improve their proposal, not to gatekeep. When something is weak, explain what would make it strong. When something is good, say so briefly and move on — don't pad the review with praise.

Helm is a community project. Assume good intent. The author invested time in writing this; respect that while being direct about problems.
