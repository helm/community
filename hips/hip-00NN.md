---
hip: "00NN"
title: "Support Git protocol for installing chart dependencies"
authors: [ "Jeff Valore (@rally25rs)", "yxxhero <aiopsclub@163.com>", "Dominykas Blyžė <hello@dominykas.com>" ]
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
- There is a [`aslafy-z/helm-git`](https://github.com/aslafy-z/helm-git) plugin available, however using plugins requires additional setup, which may not always be feasible (esp. in more complex team structures and cluster setups with advanced tooling, e.g. ArgoCD). An additional drawback is that the `Chart.yaml` does not provide a way to specify the plugin requirements.

Installing dependencies from git is an established pattern in other ecosystems even when they are registry-based, e.g. npm (Node.js), pipenv (Python), bundler (Gem) have this option - it would make sense to have the behavior replicated in Helm.

## Rationale

(TBD)

## Specification

The `Chart.yaml` should support the following format for `dependencies`:

```
dependencies:
- name: "<dependency name>"
  repository: "<protocol>://[<user>[:<password>]@]<hostname>[:<port>][:][/]<path>"
  version: "<commit-ish>"
```
where:
- `<protocol>` is one of `git`, `git+ssh`, `git+http`, `git+https`, or `git+file`.
- `<commit-ish>` is an existing reference (SHA hash, tag or branch name) on the repo.

For example:

```
dependencies:
- name: "jenkins"
  repository: "git+https://github.com/jenkinsci/helm-charts.git/charts/jenkins"
  version: "main"
```

## Backwards compatibility  

This is backwards compatible from Helm perspective - the existing formats for `dependencies` are still supported.

Charts that start using the new format will effectively be changing their minimum required Helm version, i.e. they would be introducing breaking changes and should bump their major version.

## Security implications

(TBD)

## How to teach this

(TBD)

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
