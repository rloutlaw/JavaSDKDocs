---
title: Get started with Java in Azure | Microsoft Docs
description: Update an application and push changes to App Service through local Git deployment
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

## Update the sample

Open up the `src/main/webapp/index.jsp` in your repo. Let's add a current date and time to the JSP after the "Hello from Microsoft Azure!"

```html
<body>
Hello from Microsoft Azure! 
<p>
  Current time: <%= new Date() %>
</p>
</body>
```

And add an import statement at the  of the JSP for the class we are using to get the date and time:

```html
<%@ page import="java.util.Date" %>
```

Save the file, then commit the changes and push them to Azure:

```bash
git add .
git commit -m "Add timestamp to Hello World sample"
git push azure master
```

Refresh your browser and the sample will now show a timestamp after the Hello World greeting.

>[!div class="step-by-step"]
[**Add Azure storage** &rarr;](get-started-storage.md)
