# Helm OCI MediaType Registration

The use of OCI artifacts is one of the packaging methods available in Helm. A Helm OCI artifacts is comprised of multiple component, each defined by a specific Media Type. Media Types are managed by the [Internet Assigned Numbers Authority (IANA)](https://www.iana.org) and each type should be registered with IANA so that it can not only be known by the organization, but also discoverable by end users.

This document describes the related fields associated with the [registration of a Media Type to IANA](https://www.iana.org/form/media-types).

## Helm Config

&nbsp; | &nbsp;
------ | -----
| Type Name | application |
| Tree | Vendor Tree (vnd prefix) |
| Subtype Name | `cncf.helm.config.v1+json` |
| Required Parameters | Fields as required per the Chart.yaml definition |
| Optional Parameters | Remaining fields which as defined within the Chart.yaml definition which are not noted as required |
| Encoding Considerations | Encoding considerations are identical to those specified for the "application/json" media type.  See [RFC8259](https://datatracker.ietf.org/doc/html/rfc8259). |
| Security Considerations | Similar security concerns common to all JSON content types. See [RFC 7159 Section #12](https://tools.ietf.org/html/rfc7159#section-12) for additional information. The included content as defined by the Helm chart definition may include sensitive assets including personal contact information, source code repositories or other referenceable locations. |
| Interoperability Consideration | N/A |
| Published specification | [https://helm.sh/docs/topics/charts/#the-chartyaml-file](https://helm.sh/docs/topics/charts/#the-chartyaml-file) |
| Application Usage | Internally within the Helm package manager as well as various interfacing applications |
| Fragment Identifier Considerations | N/A |
| Restrictions on Usage | N/A |
| Provisional Registrations | N/A |
| Additional Information | <ol><li>Deprecated alias names for this type: None</li><li>Magic number(s): None</li><li>File extension(s): .json</li><li>Macintosh file type code: TEXT</li><li>Object Identifiers: None</li></ol>|
| Intended Usage | Common |
| Other Information and Comments | N/A |

## Helm Content

&nbsp; | &nbsp;
------ | -----
| Type Name | application |
| Tree | Vendor Tree (vnd prefix) |
| Subtype Name | `cncf.helm.chart.content.v1.tar+gzip` |
| Required Parameters | N/A |
| Optional Parameters | N/A |
| Encoding Considerations | Binary |
| Security Considerations | No security controls are enforced by Helm. The content of a Helm package is not intended to – but may potentially – contain resources that are sensitive in nature. |
| Interoperability Consideration | N/A |`
| Published specification | None |
| Application Usage | Internally within the Helm package manager as well as various interfacing applications |
| Fragment Identifier Considerations | N/A |
| Restrictions on Usage | N/A |
| Provisional Registrations | N/A |
| Additional Information | <ol><li>Deprecated alias names for this type: None</li><li>Magic number(s): None</li><li>File extension(s): None</li><li>Macintosh file type code: None</li><li>Object Identifiers: None</li></ol>
| Intended Usage | Common |
| Other Information and Comments | N/A |

## Helm Config

&nbsp; | &nbsp;
------ | -----
| Type Name | application |
| Tree |Vendor Tree (vnd prefix) |
| Subtype Name | `cncf.helm.chart.provenance.v1.prov` |
| Required Parameters | Fields as specified within Helm provenance file definition |
| Optional Parameters N/A |
| Encoding Considerations | The utf-8 charset is always used for this type |
| Security Considerations | The contents of a Helm provenance file contains a GnuPG detached ASCII-armored signature of the Helm chart definition file as well as the definition itself. The Helm chart definition may include sensitive assets including personal contact information, source code repositories or other referenceable locations.
| Interoperability Consideration | N/A |
| Published specification | <https://helm.sh/docs/topics/provenance/#the-provenance-file> |
| Application Usage | Internally within the Helm package manager as well as various interfacing applications |
| Fragment Identifier Considerations | N/A |
| Restrictions on Usage | N/A |
| Provisional Registrations | N/A |
| Additional Information | <ol><li>Deprecated alias names for this type: None</li><li>Magic number(s): None</li><li>File extension(s): None</li><li>Macintosh file type code: Text</li><li>Object Identifiers: None</li></ol> |
| Intended Usage | Common |
| Other Information and Comments | N/A |
