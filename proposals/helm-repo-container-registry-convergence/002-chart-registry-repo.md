## Helm Charts Align With Registry Repositories
Container images are referenced with a multi-part URL:

- [**login url**] / [**namespace**] / [**repo**] : [**tag**]
- [**namespace**] / [**repo**] : [**tag** ] are equally referred to as an "image"

Different clouds implement the unique login url in a few different forms, while effectively meaning the same thing:

| Cloud | Format | eg: image |
|-----|-----|-----|
| Docker Hub | [**org**\|**user** ] / [**image**] : [**tag**] **\*1**| [contoso/marketing-fall2018-web:aab1]() |
| Azure Container Registry (acr) | [**uniqueId**].azurecr.io / [**namespace**] / [**image**] : [**tag**] | contoso.azurecr.io/marketing/fall2018/web:aab1 |
| Google Container Registry (gcr) | gcr.io/[**uniqueId**] / [**image**] : [**tag**] | gcr.io/marketing/fall2018-web:aab1 |
| AWS Container Registry (ecr) | [**aws_account_id**].dkr.ecr.region.amazonaws.com/ [**image**] : [**tag**] | 123abc456def.dkr.ecr.region.amazonaws.com/marketing/fall2018-web:aab1 |

1. Docker Hub has a default login url [docker.io]()
1. Docker Hub [**org**\|**user** ] is a [**namespace**] which is currently limted to two nodes.  
  *The [Microsoft Container Registry (mcr)](mcr.microsoft.com) has implemented a syndicated catalog representing a product hierarchy within the docker store, and looks for Docker to expand this for other partners with deep image catalogs.*

### Helm References

This proposal aligns Helm Charts with the same referencing scheme:
- [**login url**] / [**namespace**] / [**chart**] : [**version/tag**]

A user could then execute the following commands: 
- `helm login demo42.azurecr.io -u $USER -p $PWD`
- `helm push demo42.azurecr.io/marketing/shoes ./superbowl --version 1.0`
- `helm pull demo42.azurecr.io/marketing/shoes/superbowl:1.0`
- `helm upgrade superbowl-campaign demo42.azurecr.io/marketing/shoes/superbowl:1.0 --reuse values --set webui=demo42.azurecr.io/marketing/shoes/superbowl:a3nf`


### Immutable Tags and Channels
A good discussion on the benefits of immutable tags and stable channels provide the ability for a chart owner to flight a pending update to a stable tag. 

As these are seen as better together, no mutually exclusive, a separate OCI proposal will be tracked for supporting Channels across images, helm charts and other evolving artifact types, such as [CNAB](https://cnab.io)