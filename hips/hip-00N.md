---
hip: 00N
title: "Support passing multiple values files to `helm show values`"
authors: [ "Jason D'Amour <jasondamour98@gmail.com> ]
created: "2023-10-11"
type: "feature"
status: "draft"
---

## Abstract

`helm show values [CHART] --values file1.yaml -f file2.yml` should show the merged values of a chart after additional values files are merged.

## Motivation

During my deployment process, I pass N number of configuration files. Before installing the chart, I need to take some action depending on a value from one of the configuration files. I don't want to implment my own YAML merge function, as it may vary from helm's. 

## Rationale

I.E. I have 3 values.yaml files. In one of the 3 I have `ingress: true`, and in another I have `ingress: false`. I have logic in my deployment pipeline that needs to precreate some resources only if ingress is true. I could use `helm show values myOrgRepo/myChart -f f1.yml -f f2.yml -f f3.yml` to compute whether or not `ingress` is true or not after applying multiple files.


## Specification

The existing logic and flags for `--values,-f` for other commands (like upgrade/install/template) would be added to `helm show values`

## Backwards compatibility

There should be no impact to backwards compatibility, as these are new flags.

## Security implications

None

## How to teach this

Docs

## Reference implementation


## Rejected ideas

N/A. I don't know of any other ideas proposed for this feature.

## Open issues

1. https://github.com/helm/helm/issues/6772
    The accepted answer is
    > helm install [name] [chart] --dry-run --debug -f <your_values_file> and the merged list of values will be available in a section called COMPUTED VALUES
    Which is not very easy to consume programmatically. This HIP will make the same computed values printed without the manifests.

2. https://github.com/helm/helm/issues/12007
    Proposes a new standalone command just for merging value files. Also a very valid request. This HIP however merges files _with a charts default values_, which has more additional value (esp when working with remote charts).
    Was close because a truely terrible workaround was proposed, https://github.com/helm/helm/issues/12007#issuecomment-1535538080

## References

