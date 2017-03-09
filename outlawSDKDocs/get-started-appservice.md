---
title: Get started with Java in Azure | Microsoft Docs
description: Configure your Azure appservice plan and web app for the sample
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK
author: routlaw
manager: douge
ms.assetid: 7b92e776-959b-4632-8b1d-047ce1417616
ms.service: Azure
ms.devlang: java
ms.topic: reference
ms.technology: Azure
ms.date: 3/06/2016
---

# Get started with Java in Azure

## Configure Azure App Service

Use the Azure CLI 2.0 to configure the [Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/app-service-how-works-readme). In this step you will:

- create a resource group in the [West US 2 region](https://azure.microsoft.com/en-us/regions/) to manage the resources your app will use.
- create an [App Service plan](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview) 
- create a web application definition in App Service where you'll run the sample and configure it to run with Java 8 and Tomcat.

```bash
appname = AzureAppDemo$RANDOM # assign a random name to the app
az group create -n sampleResourceGroup -l westus2 
az appservice plan create --name $appname --resource-group sampleResourceGroup --sku FREE
az appservice web create --name $webappname --resource-group sampleResourceGroup --plan $webappname
az appservice web config update --resource-group sampleResourceGroup --name $webappname --java-container TOMCAT --java-version 1.8.0_73 --java-container-version 8.5
```

Verify the steps with the following command:
```bash
az appservice web browse --resource-group sampleResourceGroup --name $appname
```

Your default browser will open up to the URL where your sample will go live:

>[!div class="step-by-step"]
[**Deploy with Git** &rarr;](get-started-deploy.md)