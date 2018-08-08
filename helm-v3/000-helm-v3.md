# Helm 3 Design Proposal

This document contains a proposal for Helm 3. It assumes deep familiarity with
Helm 2, and assumes that much of Helm 2's architecture will simply carry over
without major change.

## Summary

- Tiller is gone, and there is only one functional component (`helm`)
- Charts are updated with libraries, schematized values, and the ext directory
- Helm will use a "lifecycle events" emitter/handler model.
- Helm has an embedded Lua engine for scripting some event handlers. Scripts are stored in charts.
- State is maintained with two types of object: Release and a release version Secret
- Resources created by hooks will now be managed
- For pull-based DevOps workflow, a new Helm Controller project will be started
- Cross platform plugins in Lua that only have a runtime dependency on Helm
- A complementary command to `helm fetch` to push packages to a repository

## Purpose

The design presented herein is intended to begin with the [user profiles](../user-profiles.md)
identified for helm and match as many of the [user stories](./011-user_stories.md)
as could be accommodated

## The Basic Architecture

Helm 3 is a single-service architecture. One executable is responsible for
implementing Helm. There is no client/server split, nor is the core processing
logic distributed among components.

The reference implementation of Helm 3 is a single command-line client with no
in-cluster server or controller. This tool exposes command-line operations, and
unilaterally handles the package management process.

The reference implementation has two distinct parts:

1. The command line façade, which translates commands, subcommands, flags, and
   arguments into a Helm operation
2. The Helm library, which provides the logic for executing all Helm
   operations.

By design, the Helm library _must_ be usable as a standalone library.

In Appendix A, we specify a second implementation of Helm 3 that provides a
controller-oriented model with no client. The Helm Controller provides a
CRD-based façade to receive commands, but executes them via the Helm library.

With this simplification, the following list comprises the components of Helm
v3's core implementation:

* Charts
* Chart repositories
* `helm`, the CLI
* An extension mechanism (Lua scripts stored in the chart)
* A new in-cluster CRD: Release

In this model, Tiller is removed and there are no operators or controllers that
run in-cluster.

## Command/Flag Differences from Helm 2

There are a few major (breaking) changes to the Helm CLI planned:

- The `-n` flag will be remapped across the board to `--namespace`, not `-name`.
  This is for consistency with `kubectl`, which added `-n` after Helm 2 was released.
- `helm install` will _require_ a name, unless `--generate-name` is specified. This
  inverts the default behavior for Helm 2.
- `helm serve` and `helm reset` will be removed
- `helm delete` will be renamed to `helm remove`, though `helm delete` will remain an alias.
- `helm inspect` will be renamed `helm show`, though `helm inspect` will remain as an alias.

Other changes to commands are described in their relevant sections within this document.

## Table of Contents

1.  [Charts](./001-charts.md)
2.  [Event System](./002-events.md)
3.  [State Storage](./003-state.md)
4.  [Hooks](./004-hooks.md)
5.  [Plugins](./005-plugins.md)
6.  [Repositories](./006-repositories.md)
7.  [Security Considerations](./007-security.md)
8.  [Appendix A: A Helm Controller](./008-controller.md)
9.  [Appendix B: What Is A Package Manager](./009-package_manager.md)
10. [Appendix C: Removed Sections](010-removed.md)
11. [Appendix D: User Stories](011-user_stories.md)
