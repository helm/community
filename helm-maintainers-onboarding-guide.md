# Helm Core Maintainer's Onboarding Guide

Welcome, new core maintainer! Now that you have been voted in as a new Helm maintainer, we wanted to make sure we got you off to a good start. This guide is an introduction to life as a Helm maintainer.

## Your Core Responsibilities

The primary objective of a core maintainer is to _further Helm's goal of being both an outstanding package manager and a thriving and friendly community_.

All Helm maintainers are expected to do the following:

- Positively represent Helm, Tiller, and Charts
- Hang out in the #helm-dev and #helm-users Slack channel
- Make a good-faith effort to come to the public developer's meeting
- Participate in formal Helm decision making, such as casting an official vote for new maintainers
- Uphold the Kubernetes Code of Conduct (that is, not just follow, but cultivate that as part of the project's culture)
- Participate in resolution of critical security issues as needed

## You're Encouraged To…

A successful team is not a homogenous team. Each contributor brings unique strengths and talents to the team. Because of that, we don't include things like coding PRs in the list of core responsibilities. You know your own strengths, but here are some suggested ways you can further contribute as a core maintainer:

- Triage the issue queue: This is the tactful process of responding to often-frustrated users who file bugs or request features. We take turns "owning" this responsibility week to week.
- Slack champion: Champions in Slack are maintainers who proactively work with users in #helm-users. This is quite possibly the most important way we convey Helm's friendly culture.
- Review PRs: This is the process of reading code from others, offering constructive and friendly guidance when necessary, and ultimately deciding whether it meets the project requirements.
- Release planning: We are trying to get better at planning ahead, and planning specifics. This is the process of assigning a theme to each release, and then assigning issues to that release. For example, the 2.6.0 theme is "security, stability, extensibility" and the issues assigned had to fit that theme.

## If You See Code of Conduct Violations or Bad Actors…

- A Code of Conduct violation is a case where someone appears to have transgressed the Kubernetes Code of Conduct.
- A Bad Actor is someone who is causing harm to the project (intentionally or unintentionally) through their words or actions.

If you feel like you are up to it, the best first response to either is an attempt at gentle redirection. If you do not feel like you are up for that, contact one or more of the other core maintainers and let them know about the situation. Collectively, we must make it an obligation to protect the community, and enforce standards of conduct.

If a gentle redirection is not sufficient, code of conduct violations need to be reported through the proper Kubernetes channels. The process of reporting these is documented in the code of conduct. Unless there are mitigating circumstances, Code of conduct violations reported upstream should be shared with the other Helm maintainers so they are prepared to handle the issue as it pertains to Helm while action is handled elsewhere.

## What SemVer Means to Us

If you are not familiar with SemVer (the standard), [give it a quick read](http://semver.org). We follow SemVer with a high degree of rigor. This often means that we have to tell people "no" or "wait" in order to preserve backward compatibility.

The rough application of our SemVer application goes like this: For any non-major release...

Charts:

- The Chart format MUST NOT change.
- Optional fields may be added to Chart.yaml, requirements.yaml, but chart fields MUST NOT be deleted or modified.
- Mandatory fields MUST NOT be added to Chart.yaml or requirements.yaml
- New template functions may be added, but existing functions MUST NOT be deleted.
- The default output of any command/subcommand SHOULD NOT change, except to fix bugs.
- Any modifications to chart internals MUST be backwards compatible with charts produced from Helm 2.0.0 onward.

Command Line:

- Subcommands MUST NOT be removed
- Subcommands SHOULD NOT change in meaning
- Command line flags MUST NOT be deleted
- Command line flags SHOULD NOT change in meaning

Code:

- The Protobuf files can have optional fields added, but fields MUST NOT be deleted or modified.
- Protobuf files MUST NOT have mandatory fields added
- The public API of anything in `pkg/` SHOULD NOT be modified
- Go interfaces in `pkg/` MUST NOT be modified
- The first release of a major feature addition SHOULD be hidden behind an `experimental` feature flag (see Tiller rudders)
- Critical security fixes MAY be sufficient cause to ignore the SemVer rules (but we try not to do that).

Compatibility with Kubernetes:

- A version of Helm MUST be compatible with the version of Kubernetes that it is developed against
- A version of Helm SHOULD be compatible with the previous two Kubernetes releases
- A version of Helm SHOULD be forward-compatible to the greatest extent possible

(The velocity and frequency of major changes in Kubernetes precludes us from making strong compatibility guarantees)

During a major release, all code is subject to revision, but Chart backward compatibility SHOULD be retained.

## Nuts and Bolts

This section explains (in more detail) how we do some things in Helm

### Slack and Issue Interactions

- We are friendly and empathetic.
- We understand that users are often frustrated by the time they seek our help.
- We make a good-faith effort to help where possible.
- We sometimes have to try to pull other people into a discussion to get the right answer, but we don't _demand_ that those people participate.
- Sometimes Helm cannot (and will not) fix a user's problem. We are okay admitting that.

### Issue Triaging

Core maintainers take turns triaging the issue queue. The responsibilities of a triaging run are:

- Tag all new issues
  - Bugs are tagged `bug`
  - Feature requests are tagged `feature`
  - The "default" is to tag an issue as a `question/support`
  - Anything having to do with docs are tagged `docs`
  - If the fix is simple (<10 lines of code), tag it `good first issue`
  - If a feature is deemed a Good Idea (TM), but not something we're likely to do in the near future, label it `help wanted`
- Question/Support
  - This basically means that we expect ongoing discussion
  - Sometimes we flag features and proposals as support if they seem to hinge on a misunderstanding or far-out idea
  - Sometimes we start "bugs" as "question/support" until we get sufficient info to figure out what is going on
- Bugs:
  - If it's high severity, add it to the next milestone (patch or feature)
  - Otherwise, add it to the next MINOR release
- Features:
  - If it violates the SemVer rules, mark it as `Upcoming - Major` and perhaps open a dialog on whether there is an alternate way to do this that will not break our SemVer rules
  - If it is a major feature (new subcommand, new way of doing something), ask if it can be implemented as a plugin or extension

For information on tags and their meaning, see [The CONTRIBUTING.md](https://github.com/kubernetes/helm/blob/main/CONTRIBUTING.md#issues) for the project.

During your week of triaging, periodically scan back through known recent issues to see what has been updated.

Finally, question/support, feature, proposal and non-updated pull requests _may_ be closed after 30 days of inactivity. It is up to your judgment on whether or not you close one. There is a `lifecycle/frozen` label to flag issues that should not be closed after 30 days. Additionally, Kubernetes supplies automation that will automatically mark issues as `lifecycle/stale` after 30 days, `lifecycle/rotten` after 60 days, and close them after 90 days of inactivity which are not frozen.

### PR Reviews

Helm is striving to be a properly community-driven project. We desire community-contributed code. The role of the core maintainer, in regards to the code, is to ensure that new PRs _stay true to Helm's intent, solve real problems for real users, and meet our quality guidelines_.

- Helm's Intent: Make it easy to package, share, deploy, and manage Kubernetes applications. Our guiding user paradigm is "the package manager for Kubernetes".
- Solve Real Problems for Real Users: We only want to add features to Helm if they are of benefit to many users and for non-niche use cases. This sometimes means accepting the fact that there are some things that Helm cannot do.
  - Helm is not a testbed for new ideas. We push back on cases where users say "I developed a new technology, and I want Helm to support it."
  - Many times, the way to address PRs that are niche is to suggest that they be made into plugins or extensions (rudders)
- Quality Guidelines: _We are not perfectionists. But we are enthusiasts for clarity, maintainability, and consistency._

When reviewing PRs, we need to be actively encouraging to the submitter. Strive to not leave a PR review with only "fix-it" comments. Add at least one positive note (and saying "thanks for the PR" does not count).

#### Details on How We Review PRs

If a PR passes the "Intent" and "Solves Problems" criteria above, we view our goal as getting the PR into shape and merged with minimal frustration to the submitter.

**SemVer Constraints:** Make sure a PR does not violate the SemVer constraints as explained above. This includes making sure API, ProtoBuf/gRPC, flags, commands, and formats remain compatible. Submitters often do not realize that their changes would break compatibility, so be extra gentle when explaining this.

**Coding Conventions/Idioms:** Our points of reference are [Effective Go](https://golang.org/doc/effective_go.html) and the [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments) in the Go Wiki. We follow Effective Go closely, and take the Go Code Review Comments as "decent guidelines".

We tend to avoid comments of the form "This works fine, but another way of doing it is..." unless the other way of doing it conveys clear advantage over the existing way.

**Code Style:** We have automated our style reviews. If the `gometalinter` rules in `make test-style` pass, we tend to not place any additional requirements on the user.

**Testing:** We strongly encourage PR submitters to include unit tests if they add new code. A PR should be _blocked_ if it introduces substantial new code, but that code is not covered by unit tests. The exception to this rule is if the package into which the PR is contributed is deemed "very difficult to test" by the core maintainers. (We do have a few packages like this.)

**Documentation:** If a PR contains a new feature, we tend to require accompanying documentation. This may come in the form of additional help text in the `helm` or `tiller` `--help` section, or it may require additional material in the `docs/` directory. Note that we need to be sensitive to those who do not speak English as a first language. It is okay to drop the requirement when the PR submitter does not feel they can produce quality written help.

**Scope:** We try hard to limit our comments to the things the submitter has changed. To that end, we try not to require them to make changes that are not directly related to their fix.

### Details on Merging PRs

A PR can only be merged only if:

- All tests are passing for that PR (Circle is green)
- The PR has one LGTM (small, medium) or two LGTMs (large). The GithUb Review tool is used to LGTM.
- The PR's milestone is set to the current Minor or Patch release. NEVER, EVER MERGE PRs LABELED `Upcoming - Major`
- The CLA bot has passed for the user, or you have done the due diligence to ensure that the user has signed the CLA.

Miscellaneous rules:

- If a core maintainer opened a PR, that person must be the one to either (a) merge the PR, or (b) ask another core maintainer to merge on their behalf.

### The Release Process

Corresponding to SemVer, we have three different types of release:

- Major: 1.0.0, 2.0.0, etc. Breaks compatibility with previous releases.
- Minor: 2.1.0, 2.2.0, etc. Maintains compatibility per our SemVer rules above, but may add new features, fix bugs.
- Patch: 2.1.1, 2.1.2, etc. Maintains compatibility, adds no features, but fixes bugs.

Major releases and Minor releases are done by tagging `main` with the version number, and then running the release scripts.

Patch versions are done by a more complex process. They start from the `release-X.Y` branch and cherry-pick from `main`. A release branch is forked from the last minor release, and is then maintained in parallel with the main:

```
---- v2.3.0 ---- Fix #1 --- Feature #2 --- Fix #3 --- ...  [main]
        \              \                     \
        release-2.3 ---- Fix #1 ------------- Fix #3 --- v2.3.1
```

Only fixes are pulled from `main` onto the `release-X.Y` branch. Features are not.

#### Cherry-Picking

The process of cherry-picking is done by the maintainers, typically without bringing the fix contributor back into the discussion. When a cherry-pick causes a merge conflict, the one doing the cherry-picking must decide whether to resolve the cherry-pick or whether to skip merging that issue into the release-X.Y branch.

The best practice for cherry-picking is:

1. Checkout the `release-X.Y` branch
1. Pick a commit into the branch
1. If conflict, resolve the conflict
1. Run `make test`
1. Repeat 2-4 for each necessary fix
1. Push the release-X.Y branch to your own clone
1. Create a pull request from your clone against the upstream `release-X.Y`
1. Ask other core maintainer(s) for review
