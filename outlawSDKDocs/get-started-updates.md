---
title: Get started with Java in Azure | Microsoft Docs
description: Update an application and push changes to App Service through local Git deployment
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK
author: routlaw
manager: douge
ms.assetid: 79e99ea9-f3b0-46e3-86f7-57ed14a7cbec
ms.service: Azure
ms.devlang: java
ms.topic: reference
ms.technology: Azure
ms.date: 3/06/2016
---

# Get started with Java in Azure

## Update the sample

Open up the `src/main/webapp/index.jsp` in your repo. Let's add a current date and time to the JSP after the "Hello from Microsoft Azure!"

```html
<body>
Hello from Microsoft Azure! 
<p>
  Current time: <%= new java.util.Date() %>
</p>
</body>
```

Save the changes and then rebuild and redploy:

```bash
mvn package install
```

Refresh your browser and the sample will now show a timestamp after the Hello World greeting.

>[!div class="step-by-step"]
[**Add Azure storage** &rarr;](get-started-storage.md)
