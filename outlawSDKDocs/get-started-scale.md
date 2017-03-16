---
title: Get started with Java in Azure | Microsoft Docs
description: Scale your Java web app running in Azure App Service
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK, scale
author: routlaw
manager: douge
ms.assetid: f63d6980-7ce1-4ab0-9b14-ffe34733c669
ms.service: Azure
ms.devlang: java
ms.topic: reference
ms.technology: Azure
ms.date: 3/06/2016
---

# Get started with Java in Azure

## Scale up your application

Add additional memory, CPU and storage to your application by [scaling up](https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-scale) your App Service plan. Update the
plan tier to Standard using the Azure 2.0 CLI:

```bash
# update Azure App Service plan tier to the standard tier
az appservice plan update --resource-group sampleResourceGroup --name $appname --sku S1
```

## Scale out your application

Add additional workers to your App Service plan to [scale out](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/insights-how-to-scale?toc=%2fazure%2fapp-service-web%2ftoc.json) your app deployment.

```bash
# manually update the number of workers for your apps
az appservice plan update --number-of-workers 2 --resource-group sampleResourceGroup --name $appname 
```

>[!div class="step-by-step"]
[**Monitor the sample** &rarr;](get-started-monitor.md)
