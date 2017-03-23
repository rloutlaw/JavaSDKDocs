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

Set up a Tomcat environment in Azure using the Azure CLI 2.0 to run the sample in.

```bash
# create all resources under the same resource group
az group create -n sampleResourceGroup -l westus2

# create a default webapp and resource group for future commands
appname=AzureJavaDemo$RANDOM
az configure --defaults web=$appname group=sampleResourceGroup

# create the webapp
az appservice web create --name $appname

# configure the webapp to use Tomcat 
az appservice web config update --java-container TOMCAT --java-version 1.8.0_73 --java-container-version 8.5
```

## Deploy the sample 

Deploy the sample to Azure App Service. 

```bash
# export the FTP URL and credentials for the webapp to bash
read AZSITE AZUSER AZPASS <<< $(az appservice web deployment list-publishing-profiles --query "[?publishMethod=='FTP'].{URL:publishUrl, Username:userName,Password:userPWD}" --output tsv)
export AZSITE AZUSER AZPASS

# deploy using Maven reading in the FTP information from the shell
mvn install -s az-settings.xml
```

## Verify in your browser

```bash
az appservice web browse
```
   
>[!div class="step-by-step"]
[**Deploy with Git** &rarr;](get-started-deploy.md)