---
title: Get started with Java in Azure | Microsoft Docs
description: Configure your Azure appservice plan and web app for the sample
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK
author: routlaw
manager: douge
ms.assetid: 10d02163-b62a-45dd-ad03-fce2e6023992
ms.service: Azure
ms.devlang: java
ms.topic: reference
ms.technology: Azure
ms.date: 3/06/2016
---

# Deploy a Java app to Azure App Service

## Configure App Service

```bash
# assign a random name to the sample app
appname=AzureAppDemo$RANDOM 

# create a new resoruce group for the application and its resources
az group create -n sampleResourceGroup -l westus2 

# create the App Service plan and webapp, then configure the webapp to use Tomcat 
az appservice plan create --name $appname --resource-group sampleResourceGroup --sku FREE
az appservice web create --name $appname --resource-group sampleResourceGroup --plan $appname
az appservice web config update --resource-group sampleResourceGroup --name $appname --java-container TOMCAT --java-version 1.8.0_73 --java-container-version 8.5
```

## Deploy the app via FTP with Maven

```bash
AZ_FTP_USER=<new FTP username>
AZ_FTP_PASS=<new FTP password>
az appservice web deployment user set --user-name $AZ_FTP_USER --password $AZ_FTP_PASS
mvn deploy
```

## Verify the sample

```bash
az appservice web browse --resource-group sampleResourceGroup --name $appname
```

>[!div class="step-by-step"]
[**Deploy with Git** &rarr;](get-started-deploy.md)