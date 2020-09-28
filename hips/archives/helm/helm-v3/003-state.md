# State Management

This document explains how state management works in absense of Tiller. Largely,
it is the same model with a few changes to eliminate conflicts during race conditions.

## State Storage

In Helm v3, state is tracked in-cluster by a pair of objects:

- The Release object (CRD): Represents an instance of an application
- The release version Secret: Represents a version of an instance of an application

A `helm install` creates an Application,
a Release, and a Secret. A `helm upgrade` requires an existing Application and
Release (which it may modify), and creates a new Secret that contains the new
values and rendered manifest.

### Namespacing Changes

With Tiller gone, there is no reason to default to storing data in
`kube-system`. Instead, Helm 3 will store release data _in the same namespace
as the release's destination_.

Release and release version Secrets are stored **in the same namespace as the
installed release itself**. In other words, if we install `nginx` into
namespace `foo`, then both the Release and release version Secret will also be
installed into namespace `foo`.

For example, if no namespace is specified:

```console
$ helm install -n foo bar
# everything, including Release and Secret is installed into default namespace
```

There is now only one releavant namespace, and the _tiller namespace_ goes away:

```console
$ helm install -n foo bar --namespace=dynamite
# installs release, releaseVersion, and un-namespaced charts into dynamite namespace.
```

As with Helm 2, if a resource explicitly declares its own namespace (e.g. with
metadata.namespace=something), then Helm will install it into that namespace.
But since the owner references do not hold over namespaces, any such resource
will basically become unmanaged.
> [name=Taylor Thomas]
> This could be problematic for users. I know that at Nike we are using the cross-namespace functionality and we won't be the only ones
> 
> [name=Adnan Abdulhussein]
> Should we add the concept of a un-namespaced ClusterRelease which is created for charts that are cross-namespace?
> 
> [name=Matt Butcher]
> Maybe we should confirm whether or not owner references hold across namespaces. Cuz maybe this isn't actually an issue.

### The Release Object

The release object contains information about a release, where a release is _a
particular installation of a named chart and values._ This object describes the
top-level metadata about a release.

At minimum, there are two necessary pieces of data a Release must track:

- The name of the release
- The currently deployed version (release version Secret) of this release

The release object persists for the duration of an application lifecycle, and is
the owner of all release version Secrets, as well as of all objects that are
directly created by the Helm chart. (These relationships may be represented by owner references.)

With this change, release names can now be scoped to namespace, instead of
globally scoped as they were with Helm 2.

### The release version Secret

The release version Secret ties a release to a series of revisions (install,
upgrades, rollbacks, delete). In Helm 2, revisions were merely incremental. Install
created release v1, a subsequent upgrade went to v2, and so on. And in Helm 2, 
the Release and release version Secret were collapsed into one.

For Helm 3, a Release has one or more release version Secrets associated with it.
The Release object always describes the current head release. Each release version
Secret describes just one version of that release.

An upgrade operation, for example, will create a new release version Secret, and
then modify the Release object to point to this new version. Rollback operations
can use older release version Secrets to roll back a release to a previous state.

A release version secret will have the following metadata and data fields:

| field | description |
| -------- | -------- |
| type | Always `helm.sh/release-version` |
| metadata.name | The release name plus the version ULID |
| metadata.labels.version | The ULID of the release |
| metadata.labels.release | The name of the release |
| data.userValues| The user-specified values |
| data.chartValues | The default values specified in the chart |
| data.manifest | The rendered manifest from this release |
| data.chartSource | The source of the original chart |
| data.chartName | The name and version of the original chart |

Given the above, the following operations are redescribed:

- `helm get manifest` gets `data.manifest`
- `helm get values` gets `data.userValues`
- `helm get values --all` gets `data.userValues` and `data.chartValues`
- `helm upgrade --reuse-values` uses `data.userValues`and `data.chartValues`

The owner reference of the secret will be set to the Release object.

### Versions are ULIDs, not incremented integers
Helm 2 used incremented integers for version numbering. This is changing.

Version increments are ULIDs, not SemVer or integer counters. This is to
prevent any race conditions when two revisions are pushed in near proximity.
The version incrementor need not be negotiated with ULID; each new ULID is
time-bound, but unique.
