# Security Considerations

This architecture for Helm is more secure than Helm 2 (and equally secure to
Helm Classic). The following security goals are met by this proposal:

- Helm now runs with the ID and permissions of the UserAccount or
  ServiceAccount executing the commands.
- RBAC can now be applied to each user
- On-the-wire security is as secure as the cluster is configured to be (e.g. if
  KUBECONFIG is set up for mTLS, that's what we use)
- Auditability is now a feature of the cluster, as each chart is "created by"
  the user who runs the request
- In-cluster CRDs can be secured with RBAC, too, providing access controls for
  releases.

## Signed Charts by Default

One of the weaknesses of the Helm 2 security model for chart signing is that
it relied upon the user to explictly enable and configure signing. In Helm 3,
signing will become the default.

This proposal weakens some aspects of security (such as chain of trust), but in
order to raise the overall security threshold.

### Generating PGP Keys

Upon the first run of `helm package`, a client will generate a PGP key if none
is already present. This key pair will be stored in `$HELM_HOME`.

Options will be present for importing a key from elsewhere, and the user will
be prompted to import a key from an existing chain of trust.

During `helm package`, the `prov` file will be created by default, signed by an
existing key (if present) or by the newly generated key.

### Adding Public Keys to Repositories

When a chart is added to the index using `helm index`, the public key will be
included in a designated section of the index.

```yaml
apiVersion: v2
entries:
  brigade:
  - apiVersion: v1
    appVersion: v0.12.0
    created: 2018-03-20T21:20:29.430663118Z
    description: Brigade provides event-driven scripting of Kubernetes pipelines.
    digest: 90e1cdef4303e39fb105c3a0f745a2d802bb44deaffa29f9c25933fcd53ab262
    name: brigade
    urls:
    - brigade-0.12.0.tgz
    version: 0.12.0
  - apiVersion: v1
    appVersion: v0.11.0
    created: 2018-03-20T21:20:29.430245427Z
    description: Brigade provides event-driven scripting of Kubernetes pipelines.
    digest: 40bc429edc50b66cd8ab51f419d0652b51d2aed5804fc2769b781b833168c582
    name: brigade
    urls:
    - brigade-0.11.0.tgz
    version: 0.11.0
keys:
  - |-
    -----BEGIN PGP PUBLIC KEY BLOCK-----
    Comment: GPGTools - https://gpgtools.org

    mQINBFd1LD8BEADJTqMXYG74qqmgZFjakinAJIzViJ0KmXkI+mu4S6SCwz/QuBVW
    CqnNshIfrdP3rB+LiDFybizjO9bHVTV46tZhbHGI6QWsEPnVoorZEO4v9lXGESrx
    M1EIwzvzRD2VdWaucMqLpR6YAShmiOXa2qng1OkWXFTqeq+2p8MtX19DKzhBS7r6
    +fcTODX6yyVUt1b1DO0JU+hwGJ8yNEWESp7gu7XE6rxKI+HAPhiL8oyXUIORlzAb
    p00w9I5v5KMKZcCmr09wTwvxq1DxyBYK3GGvlfWvrbK3n6iXWDYGVQf4Tcjx7A/3
    FAEriJfs8sP6lMq22b7ZsDVqJafOMeaCQTB8cmuEVHt/OKcOexpQ58AWIKyXCehf
    # ...
    PJNSLa0HO7J8vTnZOrpBFOBrneu4pjg8Z+Ozf7QgmH55BHRIzmqtiY2ZlsfxsCmI
    /4IJTxm9LpsVvzr4ZilExyFISkawKOt5P1P8B4VyORrHvn/tF8YcEliNmlXGeW+P
    KFdxAk9vWD0+F3pxXt5OVxbpAWGErKWugN6o04+yXrAI26OMI0FUAi8sWOsFbhGi
    awRB1rHuMB+3gnqAFN4lY3f9pqxh9WVXZVf08JAPheD+bb+aSlEX22sETIhylu3W
    oV8SbEZ/NmSbnbI1LNHPw7UugdVFbfxyZWX/+ll/KdFmFQclPrbQrNFs26vqtCCy
    ed3bZYsGOMVGhMojg2zdcCmJSLQN
    =bw6S
    -----END PGP PUBLIC KEY BLOCK-----
  - |-
    -----BEGIN PGP PUBLIC KEY BLOCK-----
    Comment: GPGTools - https://gpgtools.org

    mQINBFd1LD8BEADJTqMXYG74qqmgZFjakinAJIzViJ0KmXkI+mu4S6SCwz/QuBVW
    CqnNshIfrdP3rB+LiDFybizjO9bHVTV46tZhbHGI6QWsEPnVoorZEO4v9lXGESrx
    M1EIwzvzRD2VdWaucMqLpR6YAShmiOXa2qng1OkWXFTqeq+2p8MtX19DKzhBS7r6
    +fcTODX6yyVUt1b1DO0JU+hwGJ8yNEWESp7gu7XE6rxKI+HAPhiL8oyXUIORlzAb
    p00w9I5v5KMKZcCmr09wTwvxq1DxyBYK3GGvlfWvrbK3n6iXWDYGVQf4Tcjx7A/3
    FAEriJfs8sP6lMq22b7ZsDVqJafOMeaCQTB8cmuEVHt/OKcOexpQ58AWIKyXCehf
    # ...
    PJNSLa0HO7J8vTnZOrpBFOBrneu4pjg8Z+Ozf7QgmH55BHRIzmqtiY2ZlsfxsCmI
    /4IJTxm9LpsVvzr4ZilExyFISkawKOt5P1P8B4VyORrHvn/tF8YcEliNmlXGeW+P
    KFdxAk9vWD0+F3pxXt5OVxbpAWGErKWugN6o04+yXrAI26OMI0FUAi8sWOsFbhGi
    awRB1rHuMB+3gnqAFN4lY3f9pqxh9WVXZVf08JAPheD+bb+aSlEX22sETIhylu3W
    oV8SbEZ/NmSbnbI1LNHPw7UugdVFbfxyZWX/+ll/KdFmFQclPrbQrNFs26vqtCCy
    ed3bZYsGOMVGhMojg2zdcCmJSLQN
    =bw6S
    -----END PGP PUBLIC KEY BLOCK-----
```

During `helm repo add` and `helm repo update`, the user will be prompted to accept
any new (not already accepted) keys found in this file.

```
$ helm repo up
Key FINGERPRINT, attributed to USER NAME is unknown.
Accept? y, N
```

When keys are accepted, they are cached locally.

## Validating Signatures

During install, fetch, and update, Helm will automatically use the following security
policy:

- Always attempt to retrieve a provenance file
- If no provenance file, WARN
- If provenance file exists, but no public key is found, ERROR
- If procenance file does not match signing key ERROR

There will be a `--strict` verification mode that will _require_ a provenance file.

There will be an `--insecure` verification mode that bypasses all checks.

