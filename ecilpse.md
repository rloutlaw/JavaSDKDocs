---
title: Get started with Azure development in Eclipse | Microsoft Docs
description: Learn how to start working with the Azure and Java in the Eclipse IDE.
services: ''
documentationcenter: java
author: routlaw
manager: douge
editor: ''

ms.assetid: d23ec730-3046-485b-bb98-145de49bfe40
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: multiple
ms.devlang: Java
ms.topic: article
ms.date: 02/13/2016
ms.author: routlaw

---
# Get started with Azure development in Java with Eclipse

This tutorial will walk you through creating a command line tool that connects to and queries information from your Azure account.

## Create your project

1. Create a simple Maven project. Select **File** > **New** > **Maven Project**. In **New Maven Project**, select **Create a simple project** and **Use default workspace location.**
2. On the second page of **New Maven Project**, enter the following:
- Group ID: `com.<username>.azure.mgmtdemo`  
- Artifact ID: AzureMgmtDemo  
- Version: 0.0.1-SNAPSHOT  
- Packaging: jar  
- Name: AzureMgmtDemo  
Select **Finish**


## Import your dependencies

Double-click the **pom.xml** file in your project. Select the **pom.xml** tab and add the following before the `</project>` tag at the end of the file:

```XML
<dependencies>
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure</artifactId>
    <version>1.0.0-beta5</version>
</dependency>
<dependencies>
```

Eclipse will connect to Maven Central and download the Azure management libraries for Java and their dependencies.

## Set up authentication

Set up a connection key using Azure Active Directory so your app can connect to Azure.


## Connect to Azure
