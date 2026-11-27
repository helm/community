---
name: hip-author
description: "Guided authoring of Helm Improvement Proposals (HIPs). Use this skill when the user wants to write a HIP, create a proposal, draft a HIP, start a new HIP, or mentions writing a Helm Improvement Proposal. Also trigger when the user says things like 'I have an idea for Helm' or 'I want to propose a change to Helm' or 'new proposal' in the context of this repository."
---

# HIP Authoring Guide

This skill walks users through creating a Helm Improvement Proposal from an initial idea to a complete, submission-ready document.

The process has two phases: **idea refinement** (getting the concept sharp before writing) and **section-by-section drafting** (producing each section, pausing for the user to edit, then asking clarifying questions before moving on).

Read `references/hip-template.md` for the canonical template, preamble format, required sections, length recommendations, and approval requirements. Use it as the structural backbone for every HIP you help draft.

## Phase 1: Idea Refinement

Before writing anything, help the user sharpen their idea. A focused proposal is far more likely to succeed — the Helm maintainers will reject proposals that are too broad or unfocused.

### Step 1: Understand the idea

Ask the user to describe their idea in plain language. Listen for:

- What problem does this solve?
- Who experiences this problem? (chart authors, cluster operators, end users, maintainers)
- Is this a change to Helm's code (feature), a process/governance change (process), or documentation/guidance (informational)?
- Has this been discussed in [#helm-dev](https://kubernetes.slack.com/messages/C51E88VDG) on Kubernetes Slack or at the [weekly developer call](https://calendar.google.com/calendar/embed?src=s5anaqbm9kda435dnh5r8lj1l8%40group.calendar.google.com&ctz=America%2FLos_Angeles) (Thursdays 9:30am US Pacific)? If not, strongly recommend they vet the idea there first. The `#helm-dev` channel is where active development discussion happens and is the fastest way to get initial feedback on whether an idea is worth writing up as a HIP. Ideas that have been previously rejected or are out of scope for the project will be caught early, saving significant effort.

### Step 2: Scope gate

This step is critical. The single most common reason HIPs get rejected or stall is being too broad. Apply these tests:

1. **Single-concept test**: Can you explain the core of this proposal in one sentence without using "and" to join independent ideas? If not, it needs splitting.
2. **Independence test**: Does the proposal contain sub-features that could ship independently? If removing part of the proposal still leaves something valuable, the removed part should be its own HIP.
3. **MVP test**: What is the absolute simplest version of this that delivers value? Start there. Enhancements can be follow-up HIPs.

If the idea fails these tests, do NOT proceed. Instead:
- Name the independent pieces explicitly
- Help the user pick which single piece to write up first
- Explain that focused HIPs move faster through review and are more likely to be accepted

Only proceed when the user has committed to a single, focused proposal.

### Step 3: Feasibility check

Ask targeted questions about impact:

- Are there backwards compatibility concerns? If this changes existing behavior, the bar is significantly higher.
- Does this require changes across multiple Helm projects (helm, helm-www, chartmuseum, etc.), or is it scoped to one?
- Is there prior art in other package managers or CNCF projects?
- What's the rough implementation complexity? (This informs how much detail the Specification needs.)

### Step 4: Identify the type

Based on the discussion, confirm the HIP type:

- **feature** — most common; proposes a new capability or interoperability standard
- **process** — changes to how the project operates (governance, release process, tooling). Requires Helm maintainer approval rather than project maintainer approval.
- **informational** — guidance or conventions; non-binding

### Step 5: Summarize and confirm

Present a crisp 2-3 sentence summary of the proposal back to the user. This summary will become the seed for the Abstract. Ask: "Does this capture your idea? What would you change?"

Do not proceed to Phase 2 until the user explicitly confirms the idea is well-scoped and accurately summarized.

## Phase 2: Section-by-Section Drafting

Work through each section one at a time. For every section:

1. **Draft** the section content based on everything discussed so far.
2. **Present** the draft to the user with the section header, and ask them to edit directly or suggest changes.
3. **Wait** for the user to confirm or revise. If they revise, incorporate their changes.
4. **Clarify** — before moving to the next section, ask 1-2 targeted questions that will inform the upcoming sections. These questions should arise naturally from what was just written.

**Drafting principle: write each section as simply as possible while fully serving its purpose.** Each section below states what it must accomplish and gives a **target** (what is usually enough) and a **ceiling** (both drawn from real HIPs; see `references/hip-template.md` for full ranges). Aim for the target and treat the ceiling as a limit to stay under — not a quota to fill. If a section's purpose is met in fewer words, leave it shorter; reviewers reward focus, and padding buries the substance. Exceed the ceiling only when the section genuinely needs it (a detailed Specification, a breaking-change migration).

### Section order

#### 1. Preamble

Generate the YAML frontmatter. Use `hip: 9999` as the placeholder number, today's date for `created`, `status: "draft"`, and the type determined in Phase 1. Ask the user for their preferred author format (with or without email).

#### 2. Abstract

Target ~50 words (ceiling ~100). Draw from the summary confirmed in Phase 1. The abstract must be self-contained — a reader should understand what the proposal does without reading further. Avoid implementation details.

Before moving on, ask: "What existing workarounds do people use today? Understanding the current pain points will strengthen the Motivation section."

#### 3. Motivation

Target ~150 words (ceiling ~200). This section is critical — proposals without sufficient motivation get rejected outright. Use subheadings to organize, including only those that carry weight for this proposal:

- **Current Limitations** — what can't users do today?
- **Real-World Impact** — concrete examples, not hypotheticals
- **Who Is Affected** — name specific roles
- **Existing Workarounds** (optional) — what do people do today, and why is it inadequate?

Before moving on, ask: "What alternative approaches did you consider? Why did you land on this design rather than those alternatives?"

#### 4. Rationale

Target ~100 words (ceiling ~150). Explain the "why" behind key design decisions. Use subheadings for distinct decisions (e.g., "### Why X over Y"). Cover:

- Why this approach over the alternatives discussed
- How similar problems are handled in other projects if relevant
- Key tradeoffs made and why they're acceptable

Before moving on, ask: "Can you describe the technical details? What would the API, CLI interface, data structures, or behavior look like?"

#### 5. Specification

Target ~350 words (ceiling ~680 for complex features). Usually the most detailed section. For feature HIPs, it must be detailed enough for someone else to implement — but only as detailed as that requires. Include:

- Data structures, CLI flags, API shapes as appropriate
- Usage examples showing realistic scenarios (use code blocks)
- Behavior matrix if there are multiple modes or contexts
- Edge cases and how they're handled

For process HIPs, define concrete steps, timelines, roles, and decision criteria rather than code.

Before moving on, ask: "Does this change break anything for existing users? Could someone on an older version of Helm be affected?"

#### 6. Backwards Compatibility

Target ~30 words (ceiling ~90). Often a single sentence — for purely additive changes, confirming no breakage is enough. For breaking changes, be explicit about severity, migration path, and whether a major version is required.

Before moving on, ask: "Could someone misuse this feature? What's the worst case from a security perspective?"

#### 7. Security Implications

Target ~20 words (ceiling ~40). Often one line ("No security implications") for benign features. For features touching data exposure or trust boundaries, cover attack surface, risks, and mitigations.

Before moving on, ask: "If you were writing docs for this, what's the one example you'd lead with? How would you explain this to a new Helm user?"

#### 8. How to Teach This

Target ~50 words (ceiling ~70). Cover the documentation impact and nothing more:

- Documentation additions or changes needed
- The key example pattern users should follow
- How this fits into concepts users already know

#### 9. Reference Implementation

Target ~15 words (ceiling ~20). Usually one line: link to a PR, or briefly describe what the implementation will involve. If nothing exists yet, a short description is fine.

#### 10. Rejected Ideas

Target ~50 words (ceiling ~100). Capture alternatives discussed (in Phase 1 and during drafting) with brief explanations of why they were rejected. Use a bulleted list — each entry 1-2 sentences.

#### 11. Open Issues

Target ~15 words (ceiling ~50). Any unresolved questions, kept brief. Fine to submit a draft HIP with open issues — they're resolved during review.

#### 12. References

Target ~20 words (ceiling ~40). Collect all URLs, specs, and materials referenced throughout the HIP.

### Optional sections

Offer these when appropriate:

- **Scope** — when the proposal is large, explicitly stating what is NOT in scope helps focus review. Offer this after Phase 1 if the user's idea required significant narrowing.
- **Implementation Plan** — for complex multi-phase features, a phased delivery plan with milestones. Offer this if the Specification describes something that would naturally ship incrementally.

## Phase 3: Assembly and Review

After all sections are drafted and the user has reviewed each one:

1. Assemble the complete HIP document in the correct order
2. Write it to `hips/hip-9999.md` (placeholder number)
3. Remind the user to:
   - Add the HIP to the listing in `hips/README.md` once a real number is assigned
   - Submit as a PR to `helm/community`
   - Each commit must be signed off (DCO requirement)
   - The HIP needs at least 2 approvals (from project maintainers for feature/informational, from Helm maintainers for process)
4. Strongly recommend: if the user hasn't already vetted this idea in `#helm-dev` on Kubernetes Slack or at the weekly Thursday developer call, suggest they do so before or alongside submitting the PR. Getting early maintainer feedback prevents lengthy review cycles and reduces the chance of outright rejection.
