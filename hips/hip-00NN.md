---
hip: "00NN"
title: "Support Git protocol for installing chart dependencies"
authors: [ "Jeff Valore (@rally25rs)", "yxxhero <aiopsclub@163.com>", "Dominykas Blyžė <hello@dominykas.com>", "George Jenkins <gvjenkins@gmail.com>" ]
created: "2021-10-05"
type: "feature"
status: "draft"
---

## Abstract

Currently, Helm supports installing dependencies from various registries, however that requires the charts to be published there. There are use cases which are cumbersome when registries are involved, for example:

- private charts for organizations/individuals who do not have the capacity to maintain the infrastructure required to run a private registry
- testing work-in-progress charts in their umbrella charts

## Motivation

There are existing ways to achieve installation of charts without using a registry, however they are not very user-friendly and require additional tooling.

- A Helm chart repository is effectively an `index.yaml` with links to downloads. Maintaining such a repository does create a burden of scripts and automation (e.g. Github Actions). This is not always feasible for smaller projects. It also does not really offer an obvious and readily available way of testing pre-releases.
- For the testing use cases, charts can be packaged using `helm package`, however this does introduce manual steps and requires extra work to replicate in CI/CD scenarios.
- There are several plugins available to solve this problem ([`aslafy-z/helm-git`](https://github.com/aslafy-z/helm-git), [`diwakar-s-maurya/helm-git`](https://github.com/diwakar-s-maurya/helm-git), [`sagansystems/helm-github`](https://github.com/sagansystems/helm-github)), however using plugins requires additional setup, which may not always be feasible (esp. in more complex team structures and cluster setups with advanced tooling, e.g. ArgoCD). An additional drawback is that the `Chart.yaml` does not provide a way to specify the plugin requirements, which leaves it up to the consumer to figure this out.

## Rationale

Installing dependencies from git is an established pattern in other ecosystems even when they are registry-based, e.g. npm (Node.js), pipenv (Python), bundler (Gem) have this option - it would make sense to have the behavior replicated in Helm.

At least the npm and the pip ecosystems have already established a syntax of `vcs+protocol` for defining the dependency source, so it should be familiar to some users.

One alternative to consider would be to exclude defining the vcs/protocol, similar to what Go does, esp. given that Helm is built using Go. This does limit the flexibility somewhat - while Go does allow adding a VCS qualifier at the end of the URL (allowing future support for other VCSs), it does not allow specifying the protocol, which means that the users might have to override the default protocol in their VCS configuration.

## Specification

The `Chart.yaml` should support the following format for `dependencies`:

```
dependencies:
- name: "<dependency name>"
  repository: "git[+<subprotocol>]://<hostname>[:<port>][:][/]<path>"
  version: "<commit-ish>"
```
where:
- `<subprotocol>` is a protocol supported by `git clone` (e.g. `ssh`, `http`, `https`, `file`, etc).
- `<commit-ish>` is an existing reference (SHA hash, tag or branch name) on the repo.

Note that `git clone` supports having the `username` and `password` in the repository URL. The implementation of this feature should explicitly forbid that to prevent accidental credential leakage. It should throw an error if the URL contains a `username` or `password`.

For example:

```
dependencies:
- name: "jenkins"
  repository: "git+https://github.com/jenkinsci/helm-charts.git/charts/jenkins"
  version: "main"
```

When Helm is installing a dependency from git, it should:

- create a temporary directory
- clone the repo at the specified branch/tag into the temp dir
  - Helm will require a working git installation to invoke (via subprocess) in order for Helm to utilize chart's git dependencies. Helm will throw an error if git is not installed or misconfigured (e.g. credentials are not set up for private repositories).
  - for performance reasons, a shallow clone of just the latest commit of a specific branch should be performed (i.e. `git clone --depth 1 --branch <commit-ish> --single-branch --no-tags <repo-url> <temp-dir>`) 
- treat the cloned git repo similar to a `file:///path/to/temp/dir` style requirement; use `chart.LoadDir` to load that directory (which in turn applied the logic for filtering the files through `.helmignore`) and archives it to `charts/`
- delete the temp dir

## Backwards compatibility  

This is backwards compatible from Helm perspective - the existing formats for `dependencies` are still supported.

Charts that start using the new format will effectively be changing their minimum required Helm version, i.e. they would be introducing breaking changes and should bump their major version.

## Security implications

Pulling the dependencies from git may introduce additional attack surfaces, as it would need to rely on an implementation of `git` (most likely the official `git` executable), and there have been recent vulnerabilities disclosed, including Remote Code Execution (RCE).

This is something that needs to be taken into account in security conscious environments and might need to be documented for the end users. Users with high security requirements, should probably avoid using the feature and instead rely on a registry.

## How to teach this

- The documentation should note the security caveat listed above
- The documentation should provide the recommendation to prefer registries to git, if possible
- The documentation should note the implications of git being mutable with a recommendation of pinning to specific hashes
- The documentation could list the examples for various git protocols, but mention that Helm supports whatever `git clone` supports

## Reference implementation

Multiple implementation attempts available for [a discarded earlier draft of a related HIP](https://github.com/helm/community/pull/214):

- [helm/helm#11258](https://github.com/helm/helm/pull/11258)
- [helm/helm#9482](https://github.com/helm/helm/pull/9482)
- [helm/helm#6734](https://github.com/helm/helm/pull/6734)

## Rejected ideas

- An [earlier draft for solving the issue](https://github.com/helm/community/pull/214) suggested using URLs like `git://https://...`. There were comments about that approach in the reference implementations, with the suggestion to use the conventions which are already established in other ecosystems.

## Open issues

N/A

## References

- [`npm install` documentation](https://docs.npmjs.com/cli/v10/commands/npm-install) (covers the `git+[protocol]` format)
- [Python packaging documentation on version specifiers](https://packaging.python.org/en/latest/specifications/version-specifiers/) (covers the `VCS+protocol` format)
- [bundler documentation on installing gems from git repositories](https://bundler.io/guides/git.html)
