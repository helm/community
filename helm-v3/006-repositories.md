# Repositories

The following major changes will be made as it relates to working with
chart repositories in Helm 3:

- A new Capabilities API will be introduced that will allow users to
opt-in to certain repository functionality
- A new `helm login` command will be added for authenticating against a
repository provider
- A new `helm push` command will be added for pushing charts to a
repository
- The `helm serve` command will be removed
- A repository v2 specification will be defined, and the index file will be
JSON (`index.json` vs `index.yaml`)

## Capabilities API

The Capabilties API is designed to provide Helm with information such as:

1. Does this domain support actions such as login or push?
2. What supported method should the client use to perform a specific action?
3. What metadata does the client need to initiate the action?

This will allow Helm to support any number of things in the future related
to repositories without making breaking changes (e.g. remote `helm search`).

### `/.well-known/helm`

The Capabilities API will be enabled for a specific domain via
[Well-Known URI](https://tools.ietf.org/html/rfc5785) located at
`/.well-known/helm`.

*Note: This file may also serve to provide configuration for other APIs
in the future by supplying an additional top-level section.*

The response body should be valid JSON and should supply a valid,
top-level `capabilities` section. For example:

```
{
    "capabilities": {
        "login": {
            ...
        },
        "push": {
            ...
        },
        "search": {
            ...
        }
    }
}
```

Each subsection (i.e. `login`, `push`, `search`) will provide configuration
specific to the capabiltity.

For example, the following might inform Helm that this domain only
accepts token-based login:

```
...
        "login": {
            "methods": {
                "token": {
                    "loginUrl": "https://auth.site.com/oauth2/authorize",
                    "tokenUrl": "https://auth.site.com/oauth2/token"
                }
            }
        }
...
```

## The `helm login` command

A new `helm login` command will be added to Helm, responsible for
managing authentication across repository providers.

The basic usage will be:

```
helm login <url>
```

This command is intended to allow users to authenticate just one time in
order to access multiple repositories provided by the same source. 

Afterwards, running a `helm repo add` pointing to any repository URL
tied to a provider that has been successfully authenticated against will
not require additional credentials.

The exact semantics are TBD, but several auth methods will be supported,
such as:

* Basic Auth
* Cert based Auth
* Token/Bearer Auth

### Storing Credentials

Credentials obtained via `helm login` will be stored in a new `providers`
section in `$HELM_HOME/repository/repositories.yaml`.

For all provider entries, the following field is required:
* `domain` - the FQDN of the original URL specified during `helm login`

Other fields will be stored based on the authentication method used.

The following example shows a snippet of  what `repositories.yaml` might
look like with a `providers` section:
```
apiVersion: v1
generated: 2018-10-25T17:02:41.478448894-05:00
providers:
- domain: site.com
  realm: https://site.com/oauth2/helm
  expires: 2018-10-26T17:02:41.478448894-05:00
  accessToken: MTQ0NjJkZmQ5OTM2NDE1ZTZjNGZmZjI3
  refreshToken: IwOGYzYTlmM2YxOTQ5MGE3YmNmMDFkNTVk
repositories:
...
```

## The `helm push` command

A new `helm push` command will be added to Helm, responsible for uploading
chart versions to a repository.

The basic usage will be:

```
helm push <chart> <repo>
```

The exact semantics of this command are still TBD.


The most basic push method supported is referred to as `http`, and should be
present in `capabilities` in order for push to be enabled:

```
...
        "push": {
            "methods": {
                "http": {
                    "method": "POST",
                    "path": "/api/charts"
                }
            }
        }
...
```

The `"path"` field above can include templating against a limited set of chart
metatdata in order to support dynamic URLs, such as:

```
/helm/v1/repo/_blobs/${name}-${version}.tgz
```

## Removal of `helm serve`

The `helm serve` command will be removed in Helm v3.

This command started a simple webserver that provided a static repository index
based on a local directory of chart packages.

However, the majority of users hosting a static repository have done so by
uploading an index file and corresponding chart packages to some cloud storage
location accessible over HTTP, such as Amazon S3 or GitHub pages.

For users that wish to run a webserver for hosting a dynamic repository, the
Helm subproject [ChartMuseum](https://github.com/helm/chartmuseum) will be
maintained to provide this functionality.

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
