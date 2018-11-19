# Better support for CRDs

## Summary
This is an outline for how CRDs can be handled in Helm. There are at least two ways to do this currently, but both have issues. With the implementation suggested here, CRDs are getting special treatment to make sure that they are installed correctly and are updated and deleted in a safe way. CRDs will be managed as part of a release, but will be orphaned rather than deleted when a release is removed. Similar, CRDs will be adopted into a release if they already exist.

## Motivation
The current ways to handle CRDs in Helm have issues and these are known to have caused real problems for users (more details in [#4863](https://github.com/helm/helm/issues/4863)). The goal here is to handle CRDs in a way that is intuitive and flexible for chart developers, as well as easy to use and safe for helm users. The goal is that this could be introduced in Helm 2 as well as make it into Helm 3.

## Goals
* The solution should not be dependent on versioned CRDs. Even though these are in development, a solution needs to work with existing CRDs. But we would like a solution that can work with/take advantage of versioned CRDs when they become available.
* It should support existing installed charts that use CRDs, regardless of which of the existing ways to install CRDs was used.
* CRDs should never be deleted unless the user has explicitely asked for it. Since CRDs are global in a Kubernetes cluster, we can never know for sure that it is no longer in use.
* CRDs should never be updated unless the user has explicitely asked for it. Updating CRDs could render them unusable for some of the services/applications that depend on them.
* Mixing CRDs and CRs of the type in the same chart should be supported.


## Implementation
This proposal takes a very conservative approach on any changes to CRDs, but the hope is that it will not be to inconvenient. If CRDs are either only used by a single chart, or is used by many charts but rarely updated (like App CRD), updates should usually be possible without too much inconvenience for the user.

CRDs need to be handled correctly through all the phases of a Helm release:

### Install
* CRDs should be the first resource processed during installation. CRDs must be installed into the cluster before any CRs belonging to the CRD.
* If a chart contains one or more CRDs, the helm install process should for each of them, check if the CRD is already installed. 
  * If it is, Helm should check if the installed CRD is identical to the one contained in the chart. 
    * If it is, the install process can skip installing the CRD and just continue.
    * If it is different, the install process should fail, unless the user has explicitly confirmed through a command line flag that the CRD in the chart should overwrite the existing one. If a CRD is versioned, it should allow multiple different versions of the CRD to coexist in the cluster. Overwrite will only apply if a chart has a CRD with a version that already exists.
  * If it does not already exist, the CRD will be installed.
* All CRDs in the chart will be tracked as part of a release when the chart is installed
  * If the CRD was already installed, then the existing CRD will be adopted into the release.

### Delete
* When a chart is deleted, all CRDs will be orphaned, unless the user has explicitly through a command line flag decided that it should be deleted.
 * CRs that were part of the release would be deleted with the release, even if the CRD is orphaned.

### Update
* Any new CRDs included in a new version of a chart will be handled in the same way that CRDs are handled in the install flow.
* Any CRDs deleted in a new version of a chart will be handled in the same way that CRDs are handled in the delete flow.
* If a CRD is already installed in the cluster, Helm should check if the CRD in the updated chart is in any way different than the existing one
  * If it is not, then no changes are needed and the update can continue.
  * If the CRD in the updated chart is not identical to the existing CRD, the update process needs explicit approval from the user before updating the CRD.

### Rollback
Rollback would follow the same rules as update.


### Validation
Validation is harder when there are CRDs in the chart since validation of CRs will fail if the CRD has not already been installed. The simple solution here is to keep the current approach where CRDs are installed before doing validation. As a separate effort, we could see if there is a way to do client-side validation of the chart without actually installing the CRD. 


### Hooks
Hooks should run at the same time as now. Pre hooks will run after validation, but before anything is installed. Post hooks will run after all changes have been applied to the cluster.