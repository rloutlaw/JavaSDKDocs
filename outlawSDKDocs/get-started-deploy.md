---
title: Get started with Java in Azure | Microsoft Docs
description: Deploy the sample app to Azure app service
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK
author: routlaw
manager: douge
ms.assetid: 948cd16b-7b6d-4d22-bb84-69e46b90a2db
ms.service: Azure
ms.devlang: java
ms.topic: reference
ms.technology: Azure
ms.date: 3/06/2016
---

# Get started with Java in Azure

## Deploy the sample app with Git

In this step we'll upload the sample into Azure using FTP.

Create a username and password to authenticate the FTP deployment using the Azure CLI 2.0:

```bash
username=<your username>
password=<your password>
az appservice web deployment user set --user-name $username --password $password
```

(TODO: add step to get publishing profile information and export to maven when available)

The Maven project in the sample has a `deploy` step that FTPs the built sample using the environment variables created so far in the walkthrough:

```bash
mvn install
```

Verify the deployment on the web:

```bash
az appservice web browse --resource-group sampleResourceGroup --name $appname
```

>[!div class="step-by-step"]
[**Update the app** &rarr;](get-started-updates.md)