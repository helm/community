# `helm update` & `helm install` natively support remote urls

Deploying a helm chart from a remote URL is supported today, however the repo must be specified as a parameter.
```sh
helm upgrade \
    --repo=https://demo42.azurecr.io/base-charts/team-charts \
    myrelease \
    chartmuseum
```
This proposal suggests charts can be fully referenced:

```sh
helm upgrade myrelease demo42.azurecr.io/base-charts/team-charts/chartmuseum:1.0
```

Helm Charts do not need to be fully qualified.

Just as docker images don't need to be fully qualified to be used, a helm chart wouldn't require this either.

A user can build a chart locally, refer to it as `chartmusueum`, and deploy it. 

## Open Question: 

How does a chart become a fully qualified uri? 
- Does the user tag the chart, as done with docker?

  `helm tag ./chartmuseum demo42.azurecr.io/chartmuseum:1.0`
- Is the URL added when pushed?

  `helm push ./chartmuseum demo42.azurecr.io/chartmuseum:1.0`

One a chart is stored within a registry, the fully qualified uri can be used:

- `helm push ./superbowl demo42.azurecr.io/marketing/shoes/superbowl:1.0`
- `helm pull demo42.azurecr.io/marketing/shoes/superbowl:1.0`
- `helm upgrade superbowl-campaign demo42.azurecr.io/marketing/shoes/superbowl:1.0 --reuse values --set webui=demo42.azurecr.io/marketing/shoes/superbowl:a3nf`

