# Proposal: Security Mitigation File

## What is a security mitigation file and what problem it is solving?

* Security Mitigation file was developed at JFrog Community for the 
[Chartcenter](https://chatcenter.io) which is a free Helm chart central repository that was built to help the Helm community find immutable, secure, and reliable charts and have a single source of truth to proxy all the charts from one location.
* Security Mitigation file is a way for the chart maintainer to add notes which can be read on [Chartcenter](https://chatcenter.io) UI or in `security-mitigation.yaml` file directly so chart users can benefit from understanding the status of vulnerabilities present in the charts.
* It would also allow chart maintainers to be more transparent with their charts CVEs.

## Why should this be part of a Helm chart?

* It shows CVEs of docker images used in the charts.
* CVEs can be docker and chart level or both.

## Does this belong in a Helm chart? This is not even documented on helm's website

* It should belong to the chart as it covers security issues related to the chart.
* It is a new standard JFrog is trying to promote to the Kubernetes community via [Chartcenter](https://chatcenter.io), which has data for 30k+ charts, and many charts have security issues, chart users need to be aware of what they are installing into their Kubernetes clusters.

## Is it open source? Can others benefit from this file?

* Yes, it is open source.
* Cloud Native community can benefit from it, and it can become a part of the new [Open Source Security Foundation](https://openssf.org)

## Security mitigation spec

Security mitigation spec can be found [in](https://github.com/jfrog/chartcenter/blob/master/docs/securitymitigationspec.md)
