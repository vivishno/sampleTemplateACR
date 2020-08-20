# Template Workflow to deploy a ACR image to AKS using event grid

The workflows in this repo show how to deploy any image from azure container registery to azure kubernetes cluster using event grid subscription to container registery.
Based on the events trigerred the corresponding workflow will be trigerred.
- Workflow 1- subscribe to ACR -->create image and push to ACR to trigger image-push event in event grid.
- Workflow 2- the image pused to ACR is deployed to AKS based on incoming event. 

# Getting started

### 1. Prerequisites

The following prerequisites are required to make this repository work:
- Azure subscription
- Contributor access to the Azure subscription
- Resource group created on azure portal
- Azure container registery resource created in the resource group
- Azure kubernetes cluster service created in the resource group

### 2. Setting up the required secrets

#### {{AZURE_CREDENTIALS}}  ->To allow GitHub Actions to access Azure
An [Azure service principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals) needs to be generated. Just go to the Azure Portal to find the details of your resource group. Then start the Cloud CLI or install the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) on your computer and execute the following command to generate the required credentials:

```sh
# Replace {service-principal-name}, {subscription-id} and {resource-group} with your 
# Azure subscription id and resource group name and any name for your service principle
az ad sp create-for-rbac --name {service-principal-name} \
                         --role contributor \
                         --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
                         --sdk-auth
```

This will generate the following JSON output:

```sh
{
  "clientId": "<GUID>",
  "clientSecret": "<GUID>",
  "subscriptionId": "<GUID>",
  "tenantId": "<GUID>",
  (...)
}
```

Add this JSON output as [a secret](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#creating-encrypted-secrets) with the name `AZURE_CREDENTIALS` in your GitHub repository:

To do so, click on the Settings tab in your repository, then click on Secrets and finally add the new secret with the name `AZURE_CREDENTIALS` to your repository.

Please follow [this link](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#creating-encrypted-secrets) for more details. 

#### {{PATTOKEN}} ->To Allow Azure to trigger a GitHub Workflow
 We also need GH PAT token with repo access so that we can trigger a GH workflow when the training is completed on Azure Machine Learning. Add the PAT token with as [a secret](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#creating-encrypted-secrets) with the name `PATTOKEN` in your GitHub repository:


#### {{RESOURCE_GROUP}} -> Resource group where resources reside
To avoid adding resource group name in multiple places it has made a github secret.User needs to enter resource group name where resources azure container registry and azure kubernetes cluster have been created.

#### Credentials from azure container registry
Following secrets are credentials required to access azure container registry.
- REGISTRY_PASSWORD
- REGISTRY_USERNAME

```sh
# To get the above info replace {container-registry-name} with your container registryname 
az acr credential show -n {container-registry-name}
```
This will generate the following JSON output:
```sh
{
  "passwords": [
    {
      "name": "password",
      "value": "<GUID>"
    },
    {
      "name": "password2",
      "value": "<GUID>"
    }
  ],
  "username": "<GUID>"
}
```
- REGISTRY_PASSWORD any of the passwords in output can be used

### 3. Running the workflows
There are two workflow files in this repo
#### PushImage.yml -> triggers on a push to this repository
  - This workflow is used to create an event grid subscription to azure container registry events using azure_eventgridsubscriber action.
  - After subscribing to events a docker image  is built and pushed to azure container registry so that image pushed event is triggered in event grid.
#### Deploy_image.yml -> triggers when an image is pushed to subscribed acr
  - This workflow is trigerred whenever an image is pushed to acr subscribed by event grid.
  - The image pushed is deployed to azure kubernetes cluster specified in this workflow

### 4. Deployment parameters
 The deployment parameters related to deployment to aks can be changed if required by updating values in the following file-
 - charts/values.yaml
