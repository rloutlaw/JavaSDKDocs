---
title: Get started with Java in Azure | Microsoft Docs
description: Download and run the sample app you'll deploy in Azure.
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK
author: routlaw
manager: douge
ms.assetid: 760634fa-f446-4704-9935-3ac3f54ae1e5
ms.service: Azure
ms.devlang: java
ms.topic: reference
ms.technology: Azure
ms.date: 3/06/2016
---

# Download and run a sample Java app

Clone the [sample Maven project](https://github.com/rloutlaw/hello-world-java) to your system:

```bash
git clone https://github.com/rloutlaw/hello-world-java.git
```

## Run the sample locally

Build the sample and run it in a local [Jetty](http://www.eclipse.org/jetty/) container:

```bash
cd hello-world-java
mvn package jetty:run
```

Browse to http://localhost:8080/index.jsp to verify that the app is running.

>[!div class="step-by-step"]
[**Deploy to App Service** &rarr;](get-started-appservice.md)