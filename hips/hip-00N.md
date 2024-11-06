---
hip: "00N"
title: "Override AppVersion during install/upgrade"
authors: [ "Jason D'Amour <jasondamour98@gmail.com>" ]
created: "2023-09-26"
type: "feature"
status: "draft"
---

_Ah, HIPs. Where good ideas come to be killed by beaurocracy_

## Abstract

Helm offers a means to reuse charts with different versions of an app. Literally the most basic functionality of this tool is something like `helm install some-chart --set image.tag=<version>`. So since Helm has its own fields documenting the AppVersion, that should be capable of being overriden also.

## Motivation

Currently, the only way to change AppVersion is to modify Chart.yaml. That is an even bigger problem when using a remote chart from a repository.

Currently without the ability of changing appVersion on upgrade, the result of helm history is:
```
REVISION	UPDATED                 	STATUS    	CHART       	APP VERSION	DESCRIPTION
1       	Mon Jun  8 11:39:51 2020	superseded	spring-1.0.0	1.0.0      	Install complete
2       	Mon Jun  8 12:19:21 2020	superseded	spring-1.0.0	1.0.0      	Upgrade complete
3       	Mon Jun  8 12:20:51 2020	deployed  	spring-1.0.0	1.0.0      	Upgrade complete
```
Ideally running
```
helm upgrade --app-version "$MY_AWESOME_VERSION" app ./chart
```

Wold result in a more user friendly helm history, like the one below. That shows that the chart has not changed but the app had new releases.
```
REVISION	UPDATED                 	STATUS    	CHART       	APP VERSION	DESCRIPTION
1       	Mon Jun  8 11:39:51 2020	superseded	spring-1.0.0	1.0.1      	Install complete
2       	Mon Jun  8 12:19:21 2020	superseded	spring-1.0.0	1.0.2      	Upgrade complete
3       	Mon Jun  8 12:20:51 2020	deployed  	spring-1.0.0	1.0.3      	Upgrade complete
```


## Rationale

The flag should be `--set-app-version` rather than `--app-version`, because `--app-version` could potentially be used in the future to request charts which provide that AppVersion, rather than overriding AppVersion for a specific chart.

## Specification

Add new flag `--set-app-version` to helm upgrade and helm install which sets .Chart.AppVersion on the installed release. If the chart is local, then `Chart.yaml` is updated. 

## Backwards compatibility

No impact to existing workflows, its a new flag with new functionality

## Security implications

A bad actor could potentially make Helm a useful tool by accident

## How to teach this

Docs

## Open issues

- https://github.com/helm/helm/issues/8194
    - Direct parent of HIP
    - 95👍 at time-of-lock
    - 500+👍 on the whole thread
- https://github.com/helm/helm/issues/3555
    - This discussion opened the valid point of using `AppVersion` to filter for valid remote charts to install, similar to the existing `--version` flag.
    - I left the `--app-version` moniker available for this potential feature, since `--version` and `--app-version` are both _request_ flags. `--set-app-version` is a directive.
    
