# Proposal 8 - `helm search` becomes a server side query

Helm 2 relies on a local store, backed by index.yaml. As we propose transitioning to a passive cache, there's an opportunity to align search with server side semantics. 

Proposal:
- Searching the local cache:
  
  `helm search wordpress`

- Searching a specific registry:

  `helm search -r contoso.azurecr.io wordpress`

## Securing Search Results

Each cloud/vendor has implemented authentication. Each cloud/vendor has search/index solution that can limit results, based on a users permission set. 

The ability to list artifacts based on the users permission set is no different for searching images or charts. 

