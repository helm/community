# `helm update` & `helm install` support remote urls

Deploying a helm chart from a remote URL is supported today, however the repo must be specified as a parameter.
```sh
helm upgrade \
    --repo=https://demo42.azurecr.io/base-charts/team-charts \
    myrelease \
    chartmuseum
```
This proposal suggests charts can be fully referenced:

```sh
helm upgrade myrelease demo42.azurecr.io/base-charts/team-charts/chartmuseum
```

Helm Charts do not need to be fully qualified
Just as docker images don't need to be fully qualified to be used, a helm chart wouldn't require this either.

A user can build a chart locally, refer to it as chartmusueum, and deploy it. 

To push the chart, it would be "tagged", in docker terms to prepend the registry login url.