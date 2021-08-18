# arm-learning

I'm using this repo to learn Infrastructure as Code with Git Actions

## Getting Started

* Create a Github Repo online.
* Add a new action, name it then edit the yml file
Remove the default triggers and keep just the dispatch

```YML
# Controls when the workflow will run
on:
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:
```

* Clone the repo to your local machine
* make sure you have git installed and you have ran the following commands

```PowerShell
git config --global user.name "userName"
git config --global user.email "userEmail"
```

## Create a Service Principal in Azure Portal

Open the CLI in azure portal and run the following commands

```PowerShell
az account show
```

Copy the "id", as you will need it later

* This command creates a Service Principal that we will use for the GitHub Actions.

```PowerShell
az ad sp create-for-rbac --name GiHubActions --role Contributor --scopes  /subscriptions/"id_that_you_copied" --sdk-auth
```

* Go to the GitHub Repo Online, the in the settings tab click secrets
* Add the output of the Of the command that created the Service Principal.

```JSON
{
  "clientId": "the_actual_ClientId",
  "clientSecret": "the_actual_clientSecret",
  "subscriptionId": "the_actual_subscriptionId",
  "tenantId": "the_actual_tenantId",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

## Edit the workflow file

we are going to edit the armtemplatedeploy.yml file so that it is able to work with GitHub actions

* Open the file in Visual Studio

```YML
name: armTemplateDeployment

on:
  workflow_dispatch:

jobs:
  armDeploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Azure Login
        uses: Azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
```

## Create a Person Access token

To be able to commit the changes you made locally you need to create an access token.
To do this you will need to open GitHub on your browser

* Click on your profile.
* Go to settings.
* Click on Developer Options.
* Click on Personal Access Token.
* Create a Personal Access Token.
* Make sure you select repo and workflow as the scopes of the token.
* Commit the changes then push them. 
* You can now go back to GitHub in the browser and click on action, and run the action.

## Create a new arm template

We are going to create a new arm template for a resource group.

* Create a new folder named "arm-templates"
* In that folder create a new file named "resourcegroup.json"
* Add the following code and save

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "metadata": {
                "description": "Enter the location to deploy the resource to"
            },
            "defaultValue":"West Europe"
        },
        "resourceGroupName": {
            "type": "string",
            "metadata": {
                "description": "Enter the name of the resource group"
            },
            "defaultValue":"rg-githumactions"
        }
    },
    "functions": [],
    "variables": {},
    "resources": [
        {
            "name": "[parameters('resourceGroupName')]",
            "type": "Microsoft.Resources/resourceGroups",
            "apiVersion": "2019-10-01",
            "location": "[parameters('location')]",
            "dependsOn": [
            ],
            "tags": {
            }
        }
    ],
    "outputs": {}
}
```