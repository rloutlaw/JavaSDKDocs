---
title: Get started with Java in Azure | Microsoft Docs
description: Download and run the sample app you'll deploy in Azure.
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

## Get the sample app

Clone the [sample GitHub repo](https://github.com/rloutlaw/hello-world-java) to your system:

```bash
git clone https://github.com/rloutlaw/hello-world-java.git
```

## Build and verify the sample locally

Run the following to build a WAR file from the sample and run it in a local Jetty container:

```bash
cd hello-world-java
mvn package
mvn jetty:run
```

Browse to http://localhost:8080/index.jsp and verify that the app is running. Hit `Control+C` to cancel the Jetty instance when done.

>[!div class="step-by-step"]
[**Configure App Service** &rarr;](get-started-appservice.md)