# Helm Charts Use Registry Authentication

The helm client will be extended to natively support authenticating to an OCI compliant registry. By leveraging registry authentication, a user/group/service account access to one or more chart and image repositories.

The helm client authentication is cached, similar to docker credentials, but doesn't depend on docker locally installed. This allows a user, or a headless service, to authenticate with the registry, performing `pull`, `push` and `upgrade` commands. 

## Helm CLI Integrated
Different cloud providers have different methods of authentication. However, this creates a disjointed experience requiring the user to use the clouds CLI. 

The proposal enables a common authentication model that doesn't require additional plug-ins or cloud specifc helm clis: `az acr helm repo add -n demo42`

The proposal intendes to leverage OCI authentication, including:

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
