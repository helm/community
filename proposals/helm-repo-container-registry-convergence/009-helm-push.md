# Proposal 9 - `helm push` encompasses `helm package`

Tarballing a collection of helm files is a necessary detail, but not something a user should have to think about. Docker performs similar semantics for compressing images before pushing to a registry.

# helm push

A helm push command would take similar, or likely the same, parameters as `helm package`. 
The bigger question is whether charts can be referenced by their root name, without the registry, or we fully align charts with registry URLs. Aligning charts with registry URLs provides consistency with charts, minimizing concept counts. It also perpetuates the long image name challenges.

- Option 1
  
  `helm push demo42.azurecr.io/marketing/shoes ./superbowl --version 1.0`
- Option 2

  `helm push demo42.azurecr.io/marketing/shoes/superbowl:1.0`
  

## Splitting out values as a separate layer

There are two mainline scenarios for consumers using charts.

1. ISV Charts, where the user takes a public chart likely provides some values for their unique configuration
1. Custom app charts, where the developer maintains a chart to define the topology of the services, and a values file that contains the `image:tag` values for each code commit.

If a user wishes to keep these charts in their registry, and only changes the values file for each deploy, should the chart be duplicated with each push? While the chart files are small enough to not worry about size constraints of a registry, the ability to track a single chart with the variances of the values file becomes interesting. 

Once the tarballing becomes the responsibility of `helm push`, optimizations for pushing the values as a separate layer become an option as well.  The `Helm` CLI could then use the layer IDs to understand if two chart versions share the same core chart, and only differ in their values file. 
