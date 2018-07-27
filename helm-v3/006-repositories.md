# Repositories

The following changes will be made to the repository subsystem:

- The `helm serve` command will be removed
- A new `helm push` verb will be added for pushing charts to a repository
- The index file will be JSON instead of YAML

> This document is in-progress and needs input from Chart Museum.

## Pushing Charts To A Repository

Several registries have emerged with support for Helm charts including
ChartMuseum and Quay (via Application Registries) along with extras built on
top of other systems, such as object storage (e.g., see the [S3
Plugin](https://github.com/hypnoglow/helm-s3)). These registries are asking for
and could benefit from a capability to push packages from the Helm client.

To support registries there will be a `helm push` command that works in a
similar manner to `helm fetch`. It will support different systems via the
URI/URL scheme and provide a default implementation for HTTP(S). Other systems,
such as S3, can implement an uploader via a plugin for schemes Helm does not
provide an implementation for.

> [name=Matt Farina] There very well may be additional work here around
> registry search and pluggable or otherwise extendable authenication
> mechanisms.
> 
> [name=Adnan Abdulhussein] What would the default HTTP(S) implementation be?
> [name=Matt Fisher] Adnan, would you mind clarifying your comment a little?
> 
> [name=Adnan Abdulhussein] Sorry, the proposal mentions there will be a
> default implemention for `helm push` to a regular HTTP(S) chart repository.
> It's unclear to me how this would actually work.
> 
> [name=Adnan Abdulhussein] Sounds like this is something that will be fleshed
> out more in Josh's proposal, so I'll wait for that :)

## Repository v2 Specification

Helm v3 will use a second version of the repository specification. For backwards
compatibility, v1 of the specification will be supported but deprecated.

The new version of the repository specification will make the following changes
from version 1.

First, the `apiVersion` field will be incremented to `v2`. This will signal a
breaking change.

Second, the file format will be changed from YAML to JSON. JSON is more memory
efficient than YAML to parse. This is in part due to anchors and references in
YAML.

Third, there will be 2 types of files. The `index.json` file will hold the top
level details about the repository. Alongside that there will be JSON files for
each chart in the repository. These chart JSON files will contain the details
about the revisions of the chart.

The following two examples illustrate index and chart JSON files:

```json
{
  "apiVersion": "v2",
  "entries": {
    "artifactory": {
      "ref": "https://kubernetes-charts-incubator.storage.googleapis.com/artifactory.json",
      "stable": {
        "created": "2017-07-06T01:33:50.952Z",
        "description": "Universal Repository Manager supporting all major packaging formats,\nbuild tools and CI servers.",
        "digest": "249e27501dbfe1bd93d4039b04440f0ff19c707ba720540f391b5aefa3571455",
        "home": "https://www.jfrog.com/artifactory/",
        "icon": "https://raw.githubusercontent.com/JFrogDev/artifactory-dcos/master/images/jfrog_med.png",
        "keywords": [
          "artifactory",
          "jfrog"
        ],
        "maintainers": [
          {
            "email": "[redacted]",
            "name": "[redacted]"
          }
        ],
        "name": "artifactory",
        "sources": [
          "https://bintray.com/jfrog/product/JFrog-Artifactory-Pro/view",
          "https://github.com/JFrogDev"
        ],
        "urls": [
          "https://kubernetes-charts-incubator.storage.googleapis.com/artifactory-5.2.0.tgz"
        ],
        "version": "5.2.0"
      }
    }
  }
}
```

The example above is of the `index.json` file. An entry here has two properties.
The `ref` property is the location of the chart JSON file. This can be either
a relative URL path relative to the `index.json` file or an absolute path to the
file. The `stable` property contains the details about the latest stable release
of the chart.

The example below is the `artifactory.json` file referred to in the index. The
information for each version of the is listed under versions. This is in a similar
format to the repository v1 `entries` for a single chart. The different in format
is that an object with versions as keys is used instead of an array as seen in
the v1 specification.

```json
{
  "apiVersion": "v2",
  "versions": {
    "5.2.0": {
        "created": "2017-07-06T01:33:50.952Z",
        "description": "Universal Repository Manager supporting all major packaging formats,\nbuild tools and CI servers.",
        "digest": "249e27501dbfe1bd93d4039b04440f0ff19c707ba720540f391b5aefa3571455",
        "home": "https://www.jfrog.com/artifactory/",
        "icon": "https://raw.githubusercontent.com/JFrogDev/artifactory-dcos/master/images/jfrog_med.png",
        "keywords": [
            "artifactory",
            "jfrog"
        ],
        "maintainers": [
            {
            "email": "[redacted",
            "name": "[redacted"
            }
        ],
        "name": "artifactory",
        "sources": [
            "https://bintray.com/jfrog/product/JFrog-Artifactory-Pro/view",
            "https://github.com/JFrogDev"
        ],
        "urls": [
            "https://kubernetes-charts-incubator.storage.googleapis.com/artifactory-5.2.0.tgz"
        ],
        "version": "5.2.0"
    },
    "5.1.0": {
        "created": "2017-05-03T23:49:01.558Z",
        "description": "Universal Repository Manager supporting all major packaging formats,\nbuild tools and CI servers.",
        "digest": "1ce86533fca59c77e52d32138c80c332c5102557fd79f9af4afbbd7e93b9f105",
        "home": "https://www.jfrog.com/artifactory/",
        "icon": "https://raw.githubusercontent.com/JFrogDev/artifactory-dcos/master/images/jfrog_med.png",
        "keywords": [
            "artifactory",
            "jfrog"
        ],
        "maintainers": [
            {
            "email": "[redacted",
            "name": "[redacted"
            }
        ],
        "name": "artifactory",
        "sources": [
            "https://bintray.com/jfrog/product/JFrog-Artifactory-Pro/view",
            "https://github.com/JFrogDev"
        ],
        "urls": [
            "https://kubernetes-charts-incubator.storage.googleapis.com/artifactory-5.1.0.tgz"
        ],
        "version": "5.1.0"
    }
    ...
  }
}
```