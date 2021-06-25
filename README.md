# IT Conference 2021 Azure Webapps talk

You can build the talk as a [beamer](http://tug.ctan.org/macros/latex/contrib/beamer/doc/beameruserguide.pdf) pdf file using the `build.sh` file that uses [pandoc](https://pandoc.org/).

The files with `demo-app` are adapted from [Shiny Examples](https://github.com/rstudio/shiny-examples/tree/master/063-superzip-example).

## Notes for deploying as an Azure Webapp

This step-by-step guide requires the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) and the [Docker CLI](https://docs.docker.com/get-docker/) to be installed on the command line. Examples here we performed using a bash shell on Ubuntu 20.04.
### 1. Building a local container and testing it

First we build the docker image locally and test it by running it as a container.

```bash=true
# move into the demo-app directory
$ cd demo-app

# build the docker image using the Dockerfile in the current directory (.) tag it as superzip-demo
$ docker build . -t superzip-demo

# run the docker image superzip-demo forwarding port 3838 of the container to the host machine
# --rm tidies up the container after it exits preventing residual file system use
$ docker run --rm -p 3838:3838 superzip-demo
```

Navigate to [localhost:3838](localhost:3838) to view the app.


### 2. Push container image to Azure Container Registry

Next are the steps for creating your azure container registry and pushing your local docker image to your new private container registry on Azure.

```bash=
# log in the azure CLI
$ az login

# check what resource groups we got
$ az group list --output table

# create the container registry 
$ az acr create --resource-group my-resource-group \
                --name myshinyacr --sku Basic \
                --location uksouth

# log into the new acr 
$ az acr login --name myshinyacr

# update the container registry to enable an admin user (https://docs.microsoft.com/en-us/azure/container-registry/container-registry-authentication)
$ az acr update -n myshinyacr --admin-enabled true

# show as a table the login server details for our container registry
$ az acr show --name myshinyacr --query loginServer --output table

# show all local docker images
$ docker images

# tag our local image to an image set in our new container registry domain
$ docker tag superzip-demo myshinyacr.azurecr.io/superzip-demo:v1

# use docker to push (copy) the local image to the container registry
$ docker push myshinyacr.azurecr.io/superzip-demo:v1

# list the available images now present in the Azure container registry
$ az acr repository list --name myshinyacr --output table

```

### 3. Deploying the container from the container registry to our Azure web app

Next we create an app service plan and deploy our webapp using this service plan and the docker image we just pushed to our container registry.

```bash=
# create an app service plan called shiny-app-plan1 using the B1 pricing plan - https://azure.microsoft.com/en-gb/pricing/details/app-service/linux/
$ az appservice plan create -g my-resource-group \
                            -n shiny-app-plan1 \
                            --sku B1 --is-linux \
                            --location uksouth

# deploy your container as an app in the created App Service plan
$ az webapp create -g my-resource-group \
                   -p shiny-app-plan1 \
                   -n superzip-demo \
                   -i myshinyacr.azurecr.io/superzip-demo:v1 

```
