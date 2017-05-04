---
title: How to SparkPost to send email
description: Learn how to send emails from Azure through SparkPost using Node.js client library
services: app-service\web
documentationcenter: nodejs
author: rajumsys
manager: ewandennis
editor: ''
keywords: sparkpost, esp, email, smtp, bulk email

ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 05/03/2017
ms.author: rajumsys

---
# How to send email using SparkPost on Azure

## Overview
This tutorial demonstrates how to send emails via [SparkPost](https://sparkpost.com) email service on Microsoft Azure.

The example application is written in Node.js, however, same technique can be used in any language/framework.

## Prerequisites
- A Microsoft Azure Account. If you don't have an account, you can sign up for a free trial.
- A SparkPost Account
- Node.js 4x
- NPM 3x
- Git


## What is the SparkPost?
SparkPost is he world’s fastest-growing email delivery service, providing a developer friendly robust cloud API for apps and websites to send and receive email.


For more information about SparkPost, visit [https://sparkpost.com](https://sparkpost.com).

## Create a SparkPost Account
SparkPost provides 100K free emails per month including access to APIs and analytics. This free account is enough to get started however, feel free to choose [sparkpost packages] based on your sending volume requirements.


### To sign up for a SendGrid account
1. Log in to the [Azure Management Portal][Azure Management Portal].
1. In the menu on the left, click **New**.
1. Enter `sparkpost` in the **Search the marketplace** field and click on *SparkPost* in result. It'll show search results with SparkPost
    ![sparkpost-marketplace-result][sparkpost-marketplace-result]

1. Click on **SparkPost**.

    ![sparkpost-addon-create][sparkpost-addon-create]

1. Click **Create** from the panel it opened on right.
1. Fill up the signup form and click **Create**.

    ![sparkpost-addon-signup-form][sparkpost-addon-signup-form]

1. After confirming your purchase, you will see a **Deployment Succeeded** pop-up and you will see your add-on is listed in the **All resources** section.

    ![sparkpost-all-resources][sparkpost-all-resources]

1. Click on the SparkPost resource name.
    ![sparkpost-resource][sparkpost-resource]

1. Click **All Settings**, and then **Key Management**
    ![sparkpost-resource-all-settings][sparkpost-resource-all-settings]

1. In Key Management pane, it'll automatically generate one API Key. Copy the API Key, we'll use it momentarily.
    ![sparkpost-resource-key-mgt][sparkpost-resource-key-mgt]


<!--images-->
[sparkpost-search-marketplace]: ../../includes/media/sparkpost/sparkpost-search-marketplace.png
[sparkpost-marketplace-result]: ../../includes/media/sparkpost/sparkpost-marketplace-result.png
[sparkpost-addon-create]: ../../includes/media/sparkpost/sparkpost-create-azure-addon.png
[sparkpost-addon-signup-form]: ../../includes/media/sparkpost/sparkpost-addon-signup-form.png
[sparkpost-all-resources]: ../../includes/media/sparkpost/sparkpost-all-resources.png
[sparkpost-resource]: ../../includes/media/sparkpost/sparkpost-resource.png
[sparkpost-resource-all-settings]: ../../includes/media/sparkpost/sparkpost-resource-all-settings.png
[sparkpost-resource-key-mgt]: ../../includes/media/sparkpost/sparkpost-resource-key-management.png
<!--Links-->
[sparkpost packages]:  (https://azuremarketplace.microsoft.com/en-us/marketplace/apps/sparkpost.sparkpost?tab=PlansAndPrice

-----------------------

## Download the sample application
Clone the Hello World sample app repository to your local machine.

```bash
git clone https://github.com/SparkPost/azure-sparkpost-node-sample.git
```

Change to the directory that contains the sample code.

```bash
cd azure-sparkpost-node-sample
```

## Run the application locally

Run the application locally by opening a terminal window and using the `npm start` script for the sample to launch the built in Node.js http server.

```bash
SPARKPOST_API_KEY=<api_key> npm start
```

Replace `<api_key>` with the API Key we've just created in the previous step.

Open a web browser, and navigate to the sample.

```bash
http://localhost:8080
```

![run-locally][run-locally]


> [!TIP]
> To run the application on different port prefix the command with desired port like
> ```bash
> PORT=3000 SPARKPOST_API_KEY=<API_KEY> npm start

## Log in to Azure

We are now going to use the [Azure CLI](azure-cli) in terminal to create the resources needed to deploy our above sample Node.js application to Azure. Log in to your Azure subscription with the [az login](/cli/azure/#login) command and follow the on-screen directions.

```azurecli
az login
```

## Create a Deployment User

We'll need a deployment user in order to authenticate the deployment. [Creation of deployment user](/cli/azure/appservice/web/deployment/user#set) is a one-time process.


```azurecli
az appservice web deployment user set --user-name <username> --password <password>
```

Supply your desired values for `<username>` and `<password>` and take a note of them as they will be used in a steps below.

## Create a Resource Group

Create a [resource group] with the [az group create](/cli/azure/group#create) command.

```azurecli
az group create --name testResource --location westus
```

This created a new resource group named `testResource` in West US location.

## Create an App Service Plan

We'll now create a Linux-based App Service Plan with the [az appservice plan create](/cli/azure/appservice/plan#create) command.

```azurecli
az appservice plan create --name sparkpost_nodejs --resource-group testResource --sku S1 --is-linux
```

The above commands created an App Service Plan on Linux Workers named `sparkpost_nodejs` using the **Standard** pricing tier.

Upon success, it'll return a response similar to following.

```json
{
  "adminSiteName": null,
  "appServicePlanName": "sparkpost_nodejs",
  "geoRegion": "West US",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/6c6fe2eb-ac4c-4944-a320-203a08a12c1a/resourceGroups/testResource/providers/Microsoft.Web/serverfarms/sparkpost_nodejs",
  "kind": "linux",
  "location": "West US",
  "maximumNumberOfWorkers": 10,
  "name": "sparkpost_nodejs",
  "numberOfSites": 0,
  "perSiteScaling": false,
  "provisioningState": "Succeeded",
  "reserved": true,
  "resourceGroup": "testResource",
  "sku": {
    "capabilities": null,
    "capacity": 1,
    "family": "S",
    "locations": null,
    "name": "S1",
    "size": "S1",
    "skuCapacity": null,
    "tier": "Standard"
  },
  "status": "Ready",
  "subscription": "6c6fe2eb-ac4c-4944-a320-203a08a12c1a",
  "tags": null,
  "targetWorkerCount": 0,
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
}
```

## Create a web app

Now we'll create a web app within the `sparkpost_nodejs` App Service plan. The web app gives us a hosting space to deploy our code as well as provides a URL for us to view the deployed application. Use the [az appservice web create](/cli/azure/appservice/web#create) command to create the web app.

```azurecli
az appservice web create --name <app_name> --resource-group testResource --plan sparkpost_nodejs
```

Replace `<app_name>` with your own app name. As this value will be used as the default DNS site for the web app, it needs to be unique across all apps in Azure. However, you can later map any custom DNS entry to the web app.

A successful request will respond with something like following.

```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "enabled": true,
  "enabledHostNames": [
    "<app_name>.azurewebsites.net",
    "<app_name>.scm.azurewebsites.net"
  ],
  "gatewaySiteName": null,
  "hostNameSslStates": [
    {
      "hostType": "Standard",
      "name": "<app_name>.azurewebsites.net",
      "sslState": "Disabled",
      "thumbprint": null,
      "toUpdate": null,
      "virtualIp": null
    },
    {
      "hostType": "Repository",
      "name": "<app_name>.scm.azurewebsites.net",
      "sslState": "Disabled",
      "thumbprint": null,
      "toUpdate": null,
      "virtualIp": null
    }
  ],
  "hostNames": [
    "<app_name>.azurewebsites.net"
  ],
  "hostNamesDisabled": false,
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/6c6fe2eb-ac4c-4944-a320-203a08a12c1a/resourceGroups/testResource/providers/Microsoft.Web/sites/<app_name>",
  "isDefaultContainer": null,
  "kind": "app",
  "lastModifiedTimeUtc": "2017-05-04T19:43:55.150000",
  "location": "West US",
  "maxNumberOfWorkers": null,
  "microService": "false",
  "name": "<app_name>",
  "outboundIpAddresses": "13.93.220.109,40.118.160.111,13.64.239.21,13.91.92.146,13.93.200.235",
  "premiumAppDeployed": null,
  "repositorySiteName": "<app_name>",
  "reserved": true,
  "resourceGroup": "testResource",
  "scmSiteAlsoStopped": false,
  "serverFarmId": "/subscriptions/6c6fe2eb-ac4c-4944-a320-203a08a12c1a/resourceGroups/testResource/providers/Microsoft.Web/serverfarms/sparkpost_nodejs",
  "siteConfig": null,
  "slotSwapStatus": null,
  "state": "Running",
  "suspendedTill": null,
  "tags": null,
  "targetSwapSlot": null,
  "trafficManagerHostNames": null,
  "type": "Microsoft.Web/sites",
  "usageState": "Normal"
}
```

You should be able to browse your newly created Web App.

```
http://<app_name>.azurewebsites.net
```

![default-app][default-app]

This is default web app in Azure. We'll now configure it to use Node.js.

## Configure to use Node.js

Use the [az appservice web config update](/cli/azure/appservice/web/config#update) command to configure the Web App to use Node.js. We'll use Node `v4.4.7` for this tutorial.

```azurecli
az appservice web config update --linux-fx-version "NODE|4.4.7" --startup-file process.json --name <app_name> --resource-group testResource
```

## Configure local git deployment
```azurecli
az appservice web source-control config-local-git --name <app_name> --resource-group testResource --query url --output tsv
```

It'll return an git repo URL. Copy it as we'll use this in next step.

```bash
https://<username>@<app_name>.scm.azurewebsites.net:443/<app_name>.git
```

## Add Azure as Git Remote

```bash
git remote add azure <git_repo_url>
```

## Push to Azure from Git
Now that we've added git remote, we can simply push codes to this origin.

```bash
git push azure master
```

Once prompted, enter the password we've used during creation of deployment user.

During deployment, Azure App Service will communicate it's progress with Git.

```bash
Counting objects: 12, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (9/9), done.
Writing objects: 100% (12/12), 11.21 KiB | 0 bytes/s, done.
Total 12 (delta 0), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '1e0352efa1'.
remote: Generating deployment script.
remote: Generating deployment script for node.js Web Site
remote: Generated deployment script files
remote: Running deployment command...
remote: Handling node.js deployment.
remote: Kudu sync from: '/home/site/repository' to: '/home/site/wwwroot'
remote: Copying file: '.gitignore'
remote: Copying file: 'README.md'
remote: Copying file: 'app.yaml'
remote: Copying file: 'package.json'
remote: Copying file: 'server.js'
remote: Copying file: 'yarn.lock'
remote: Deleting file: 'hostingstart.html'
remote: Ignoring: .git
remote: Copying file: 'views/index.pug'
remote: Using start-up script server.js from package.json.
remote: Node.js versions available on the platform are: 4.4.7, 4.5.0, 6.2.2, 6.6.0, 6.9.3, 6.10.2.
remote: Selected node.js version 6.10.2. Use package.json file to choose a different version.
remote: Selected npm version 3.10.10
remote: npm WARN deprecated pug@0.1.0: Please update to the latest version of pug, at time of writing that is pug@2.0.0-alpha6
remote: .
remote: npm WARN deprecated pug-loader@0.0.0: Please use pug-load for pug-loader@<=1.0.2.
remote: ..............................................................................
remote: azure-sparkpost@0.0.1 /home/site/wwwroot
remote: npm WARN azure-sparkpost@0.0.1 No repository field.
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Deployment successful.
To https://hello-sparkpost.scm.azurewebsites.net/hello-sparkpost.git
 * [new branch]      master -> master
```

<!--images-->
[run-locally]: ../../includes/media/sparkpost/sparkpost-run-app-locally.png
[default-app]: ../../includes/media/sparkpost/sparkpost-default-app.png
<!--links-->
[azure-cli]: https://docs.microsoft.com/en-us/cli/azure/overview
[resource group]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview
