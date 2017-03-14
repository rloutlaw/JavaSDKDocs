---
title: Get started with Java in Azure | Microsoft Docs
description: Install the Azure CLI for your platform and log in to your Azure account
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK
author: routlaw
manager: douge
ms.assetid: ddecffcb-29e2-4fa5-9934-3e5f19f8fbdf
ms.service: Azure
ms.devlang: java
ms.topic: reference
ms.technology: Azure
ms.date: 3/06/2016
---

# Get started with Java in Azure

## Set up your environment

[Install the Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2) for your platform. You'll use the CLI to create the Azure resources your app will need to run.

Once you have the Azure CLI 2.0 installed, login to Azure and verify your installation:

```bash
az login
az account show
```   

If the CLI is set up correctly, you'll see output like this from `az account list`:
```json
{
  "environmentName": "AzureCloud",
  "id": "ac6c882c-52b3-4dfb-99d3-bc76026cb6da",
  "isDefault": true,
  "name": "Visual Studio Enterprise",
  "state": "Enabled",
  "tenantId": "6c3b4f5c-a18b-455c-87b0-d9d26521a78c",
  "user": {
    "name": "frank@frabrikam.com",
    "type": "user"
  }
}
```

>[!div class="step-by-step"]
[**Get the sample app** &rarr;](get-started-sample.md)
