# Security Mitigation Notes

Security Mitigation Notes are a way for you as the chart maintainer to add notes on our UI that users can read to understand the status of vulnerabilities. Your high severity vulnerabilities are hidden by default to give you time to mitigate. 

## Opt-in to show high severity vulnerabilities

**Once you add mitigation notes to any high severity vulnerability using the security-mitigation.yaml (outlined below), you are opting-in for us to make all of your vulnerability information available on the UI.**

![Mitigation](https://github.com/jfrog/chartcenter/blob/master/docs/mi1.jpg?raw=true)

You can use the spec below to fill out the [security-mitigation.yaml](https://github.com/jfrog/chartcenter/blob/master/docs/security-mitigation.yaml) file to get these notes on the security tab of your chart on ChartCenter. Once we've received the filled out security-mitigation.yaml, you will receieve a confirmation and your mitigation summary and indvidiual CVE notes will be available on the UI for everyone.

Please note that high severity vulnerabilities are hidden on the UI be default, **but once you include your security-migitation.yaml with any high CVE tagged**, your high vulnerabiliteis will be activated and we will publish details of high vulnerabilities on the security tab as well.

**Once you add a single CVE note on a high vulnerability and send us the file, you are opting in for us to activate details for all high vulnerabilities.**

This spec will walk you through how to the  should be filled out.

# Here is the Spec:

You can get a copy of the [security-mitigations.yaml](security-mitigation.yaml) file here.

![Example](https://github.com/jfrog/chartcenter/blob/master/docs/screen4.png?raw=true)

Security mitigation provides the ability for producers to specify mitigation notes for security issues associated with their Helm chart with their consumers.

These mitigation notes will appear on the security tab of your Helm chart on ChartCenter.

The security mitigation spec supports 3 use cases:
* Ability for producers to provide overall and/or CVE specific mitigation information. 
* Ability for producers to point security to a mitigation website that is hosted externally on a wiki / webpage.
* Ability for producers to point to externally hosted security-mitigation.yaml file.

Here are the fields:

| Field  | Description | Type |
| ------------- | ------------- | ---- |
| summary  | Overall mitigation summary that applies to all chart versions  | text |
| securityAdvisoryUrl | Link pointing to a mitigation information hosted externally such as wiki, web page, etc. | url |
| useMitigationExternalFile | true means security-mitigation.yaml is hosted somewhere else. false means the content of the current file represents security mitigation information. Default value: false | true/false | 
| mitigationExternalFileUrl | If useMitigationExternalFile is set to true, then this parameter points to a url of externally hosted security-mitigation.yaml | url | 
| mitigations: cves | List of CVEs for which mitigation notes are being provided. | CVE-YYYY-NNNN | 
| mitigations: cves: affectedPackageUri | Indicates package Uri for which the security mitigation is provided. Currently we support only two package uri: Docker docker://docker.io/bitnami/postgres Helm helm://artifactory | uri | 
| mitigations: cves: affectedVersions | SemVer Constraint from Masterminds/semver as used on Chart.yaml for kubeVersion specifying which versions should use the mitigation information. | Example: > 1.2.x || < 2.5.8 | 
| mitigations: cves: description | Mitigation notes at CVE level. | text description | 

## Examples
Example 1: Ability for producers to provide overall and/or CVE specific mitigation information. 

```
schemaVersion: v1
summary: This chart is secure
securityAdvisoryUrl:
mitigationExternalFileUrl:
mitigations:   
    cves: 
        CVE-1234
        CVE-5432
    affectedPackageUri: 
    affectedVersions: > 1.2.x || < 2.5.8
    description: This security mitigation information for CVE-1234 & CVE-5432 applies to the specified affectedVersions of charts.
    cves: 
        CVE-3456
    affectedPackageUri: 
    affectedVersions: 6.x || 7.x
    description: This security mitigation information for CVE-3456 applies to application versions spe & CVE-5432 applies to the specified affectedVersions of the application.
```


Example 2: Ability for producers point security mitigation information that is hosted externally on wiki / webpage.

```
schemaVersion: v1
summary: Artifactory helm chart is secure and details are mentioned in the security advisory page. 
securityAdvisoryUrl: https://jfrog.com/artifactory/security-advisory.html
```

Example 3: Ability for producers to point to externally hosted security-mitigation.yaml file.

```
schemaVersion: v1
summary: Artifactory helm chart is secure and details are mentioned in the security advisory page. 
securityAdvisoryUrl: https://jfrog.com/artifactory/security-advisory.html
mitigationExternalFileUrl: https://jfrog.com/artifactory/security-mitigation.yaml
```

## Presence of security-mitigation.yaml also represents intent for producers to be transparent with all high severity issues with their consumers. 

**Explanation:** once a mitigation note is provied for 1 high CVE, all high CVEs will be activiated and made public in the UI. 

**Example 1:** Let’s assume that **application A** has 5 high severity issues. If mitigation information is provided for 1 CVE that has high severity, then all 5 security issues will be made public on the UI.
