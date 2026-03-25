---
hip: 9999
title: "Moving non-Action packages to be internal"
authors: [ "George Jenkins <gvjenkins@gmail.com>" ]
created: "2020-08-13"
type: "feature"
status: "draft"
helm-version: "4"
---

## Abstract

This HIP proposes moving non-Action packages which are unlikely to be used by Helm SDK clients to be internal/non-public.

## Motivation

A significant awkwardness with Helm 3, was that all public packages had to remain API compatible.
And a large portion of Heln's implementation code was directly exposed as public packages.
Having no seperatation between API interface and implementation frequently made it difficult to introduce changes to Helm's codebase.
Even if public APIs were unlikely to be used by external SDK users.

## Rationale

Moving packages to be internal should allow more flexibility for Helm to make changes in the the future.
Functionality that is desired to be exposed to public can be exposed via public API specific wrappers.
Reducing the friction between improving/extending/modifying/extending Helm's functionality.
And providing stable API guarentees.

## Specification

The following packages could be moved to be `internal`:

```
pkg/chart
pkg/chartrelease
pkg/chartutil
pkg/engine
pkg/gates
pkg/ignore
pkg/kube
pkg/kube/fake
pkg/lint
pkg/lint/rules
pkg/lint/support
pkg/plugin
pkg/plugin/cache
pkg/plugin/installer
pkg/postrender
pkg/provenance
pkg/pusher
pkg/registry
pkg/release // Some external usage, replacement public API creation should be examined
pkg/releaseutil
pkg/repo
pkg/repo/repotest
pkg/storage
pkg/storage/driver
pkg/strvals
pkg/time
pkg/uploader
```

The following packages should remain as public facing packages:
```
pkg/action
pkg/cli
pkg/getter // Implementations should be moved to be internal
pkg/helmpath
pkg/chart/loader
pkg/downloader
```

## Backwards compatibility

From a search in public github, the list to move to internal seems to either be minimally (indicating a limited pulic API could be created, or public use deprecated) or unused.
As such, moving these packages to internal will likely have minimal/insignificant impact to the community.
If required, packages could easily be returned to being public should the community request even once Helm 4 is released.

## Rejected ideas

Moving packages which provide core SDK functionality, such as `pkg/action`.
Or packages which seem to have significant external usage.
