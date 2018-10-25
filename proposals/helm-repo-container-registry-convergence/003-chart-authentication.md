# Helm Charts Use Registry Authentication

Enabling a registry to assign a user/group/service account access to one or more chart and image repositories

Client authentication is cached, similar to docker credentials, but doesn't depend on docker locally installed

## Helm CLI Integrated
Different cloud providers have different methods of authentication. However, this creates a disjointed experience requiring the user to use the clouds CLI and tool CLI. 
The proposed goal here is to enable a common authentication model that doesn't require additional plug-ins.

It's intended to support a common API on Helm Chart repositories to support:

- Basic Auth
- Token Auth
- Cert Based Auth
- 2 Factor Auth

### Proposed Experience
To create a Helm native experience, we're proposing:
```sh
# basic auth
helm login [registryURL] -u $USER -p $PASSWORD
# token
helm login [registryURL] --token $TOKEN 
# cert
helm login [registryURL] --cert --certPath $OPTIONAL_CERT_PATH
```
Example:
```sh
helm login demo42.azurecr.io -u -p 
```
