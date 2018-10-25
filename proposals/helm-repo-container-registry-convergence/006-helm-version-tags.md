# Proposal 6 Helm Chart Versions merge with docker tags

2.0 Helm Charts use semantic versioning. Semantic versioning is far more predictable and structured then docker tags. 

Proposal:

1. [Maintain semantic versioning](#proposal-8-1)
2. Align with : notation to differentiate domains (.), namespaces (/) and file name extensions. 
3. Maintain "latest" concept, which also aligns with the docker manifest concept to have two tags equate tot the same chart.
### 1 Maintain Semantic Versioning <a id="proposal-8-1"></a>
To avoid the open form, difficult to parse docker tagging format, semantic versioning will continue to be supported. 
However, similar to stable tags, including :latest, helm charts would provide versioned, stable releases.
Helm Charts would leverage manifests that enable two or more "versions" to reference the same chart.

- `helm pull demo42.azrecr.io/base-charts/team-charts/chartmuseum:1.2.1`
- `helm update myDeployment demo42.azrecr.io/base-charts/team-charts/chartmuseum:1.2.1`





