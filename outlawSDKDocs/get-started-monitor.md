---
title: Get started with Java in Azure | Microsoft Docs
description: Add monitoring and telemetry to your Java web application using AppInsights
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK, monitoring
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

## Monitor the sample through the Azure portal

Azure App Service uses the [Azure monitor] to create visualizations of key performance metrics in your app with no additional configuration: 

- Live HTTP request volume, error volume, and response times
- CPU and memory statistics for each instance
- Network usage 

## View performance metrics on the Azure portal

0. Open the [Azure portal](https://ms.portal.azure.com)
0. Select **App Services** from the left hand side of the portal.
0. Select the App Service entry you created for the sample app.
0. Scroll down to **Monitoring** in the list of actions. The visualizations will appear under **Metrics per instance (Apps)** and **Live HTTP traffic**. 

Instrument your apps with [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-java-get-started) to gather additional performance details and track user behavior and usage.