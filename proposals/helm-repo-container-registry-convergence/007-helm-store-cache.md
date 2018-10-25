# Proposal 7 - Helm Chart Cache
To support multiple registries, with nested repositories, a folder cache, with matching hierarchy is proposed:
`demo42.azrecr.io/base-charts/team-charts/chartmuseum:1.2.1` would be represented on disk as:

```
└── demo42.azrecr.io
    └── base-charts
        └── team-charts
            └── chartmuseum
                └── 1.2.1
```
### Helm Chart Cache

The helm chart store is typically a local store. To pull a public chart, make changes, then push the chart to a private helm repository, the local store can get out of sync.