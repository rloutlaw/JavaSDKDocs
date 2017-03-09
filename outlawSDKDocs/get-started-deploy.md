---
title: Get started with Java in Azure | Microsoft Docs
description: Deploy the sample app to Azure app service
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

## Deploy the sample app with Git

In this step we'll push the sample app in the local repo cloned in the previous step into Azure. 

Give the app a username and password to authenticate deployments, then configure it for local Git deployment:

```bash
cd ..
username=<your username>
password=<your password>
az appservice web deployment user set --user-name $username --password $password
url=$(az appservice web source-control config-local-git --name $webapp \
--resource-group sampleResourceGroup --query url --output tsv)
cd hello-world-sample
git remote add azure $url
```

Push the sample to Azure using the new remote. Enter the password for the deployment credential you set up when prompted. App Service will build and deploy the app in the local repo.

```bash
git push azure master
```

Verify the deployment on the web:

```bash
az appservice web browse --resource-group sampleResourceGroup --name $appname
```

>[!div class="step-by-step"]
[**Update the app* &rarr;](get-started-updates.md)