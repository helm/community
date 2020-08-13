# Proposal: Security Mitigation Standard

## What is a security mitigation standard and what problem it is solving?

* There was no easy way for chart authors to share security mitigation information, chart users not able to read about CVEs present in charts. To fix this broken communication between authors and consumers the security mitigation standard was developed.
* Proposed [Security Mitigation file](security-mitigation.yaml) is a way for the chart maintainer to add security mitigation notes for CVEs.
* It would also allow chart maintainers to be more transparent with their charts CVEs.

## Is it open source? Can others benefit from this file?

* This is an open standard.
* [Chartcenter](https://chartcenter.io) currently captures the data and other hubs as ArtifactHub, Kubapps can proxy this information from there or use `security-mitigation.yaml` file directly from charts as well and use that information to evolve their UIs.
* Cloud Native community can really benefit from it, and it can become a part of the new [Open Source Security Foundation](https://openssf.org) too.

## Why should this be part of a Helm chart?

* It shows CVEs of docker images used in the charts.
* CVEs can be docker or chart level or both.
* It should be part to the chart as it covers security issues related to the chart.

## Security mitigation spec

Security mitigation spec can be found [here](securitymitigationspec.md)
