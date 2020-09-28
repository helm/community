# Proposal: OCI Support Updates

A couple of the maintainers recently met regarding how to move forward with Helm's experimental OCI support. This documents the conclusions reached.

Note: the existing support for OCI registries is documented [here](https://helm.sh/docs/topics/registries/).

## 1. Implement `Getter` and introduce `Pusher`

As of yet, the code related to registry support has been written standalone (see `internal/experimental/registry`).

The act of downloading a chart from an OCI registry should mimic what is already possible using downloader plugins. In theory, using a protocol prefix such as `oci://` in any place where chart repos are referenced should work just by implementing a `Getter` for OCI.

Currently there is no Helm-specific way to upload chart packages. Using the same model of `Getter`, a new interface called `Pusher` should be introduced, with a single built-in implementation (OCI). This opens the door for plugins to implement their own upload mechanism, and for Helm to add a new top-level upload command such as `helm push`.


## 2. Support for provenance files

Helm currently has the ability to verify package signatures, assuming the presence of a file suffixed with `.prov` sitting next to a chart `.tgz` in a repository.

Support for this method of signature validation should be carried over into OCI storage. As the format of the OCI manifest is custom to Helm, Helm can also choose to modify the resulting manifest on upload when a chart is being signed (e.g. `helm push --sign`).

As far as the low-level details, the `.prov` file will simply be stored as a second layer on the manifest. If running `helm pull --verify oci://...`, the second layer (layer at index 1) will be assumed to be the provenance file. The first layer is always the chart `.tgz` itself.

Here is an example of what a manifest might look like with a provenance file attached:

```
{
  "schemaVersion": 2,
  "config": {
    "mediaType": "application/vnd.cncf.helm.config.v1+json",
    "digest": "sha256:8ec7c0f2f6860037c19b54c3cfbab48d9b4b21b485a93d87b64690fdb68c2111",
    "size": 117
  },
  "layers": [
    {
      "mediaType": "application/tar+gzip",
      "digest": "sha256:1b251d38cfe948dfc0a5745b7af5ca574ecb61e52aed10b19039db39af6e1617",
      "size": 2487
    },
    {
      "mediaType": "application/pgp-signature",
      "digest": "sha256:3e207b409db364b595ba862cdc12be96dcdad8e36c59a03b7b3b61c946a5741a",
      "size": 643
    }
  ]
}

```

If `application/pgp-signature` is not appropriate based the contents of a `.prov` file (e.g. it does not meet the IANA requirements), a custom media type will be used such as `application/vnd.cncf.helm.provenance.v1`.

## 3. Chart versions == OCI reference tags

To keep things simple, the version of a chart will be 1-to-1 with the tag used on registry references. Arbitrary tags will not be supported for initial release. As time goes on, support for arbitrary tags may be reintroduced.

This also means that tags are no longer necessary to be provided on the command-line in the form `<ref>:<tag>`. Some users feel strongly to keep this to match the UX of Docker and other tools, so perhaps this feature is kept and treated the same as using the `--version` flag. Thoughts?

The latest sematic version is able to be determined from a registry by listing all tags. To specify a specific tag/version (or if a registry does not support tag listing), users can provide `--version <version>`.

## 4. Chart names == OCI reference basenames

Again, to keep things simple, the basename (the last segment of the URL path) on a registry reference should be equivalent to the chart name. 

For example, given a chart with the name `pepper` and the version `1.2.3`, users may run a command such as the following:

```
$ helm push pepper/ oci://r.myreg.io/mycharts
```

which would result in the following reference:

```
oci://r.myreg.io/mycharts/pepper:1.2.3
```

By placing such restrictions on registry URLs and tags, Helm users are less likely to do "strange things" with charts in registries. Again, support for custom tags and basenames could be supported in the future via options such as `--oci-basename` after we first stabilize the feature set.

## 5. `helm chart` is removed, integrated into rest of CLI

Wherever possible, the subcommands provided by the new `helm chart` command should be integrated into existing Helm CLI commands.

For example, instead of `helm chart save`, we may use something like `helm package --cache-oci` which would store it into the OCI cache. We may also choose to do away with the OCI cache entirely.

`helm chart pull` should be baked into `helm pull`, `helm chart push` should be `helm push` (new). `helm chart remove` and `helm chart export` could potentially be removed if we no longer have an OCI cache. Thoughts on this?

Regarding the other new command, `helm registry`, this should probably be kept as is. This manages auth against OCI registries. `helm login` might indicate a login against a Kubernetes cluster, but this is another option.


## 6. Experimental until clear messaging from OCI

Considering that the rest of the items above are implemented and resolved, the OCI feature set will not be made generally available until there is clear messaging from the Open Container Initiative (OCI) regarding the way that Helm is using registries.

It is somewhat controversial that the APIs originally intended for container images are being reused to store things that are not container images. Additionally, registries like Docker Hub do not yet support this.

The technical details of how Helm is using the OCI distribution spec must be validated in writing, in a document approved by OCI, clearly stating that this is the correct way to publish an arbitrary artifact. Only after this will these features be taken out of experimental mode and made generally available.
