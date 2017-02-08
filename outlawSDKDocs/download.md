---
title: Download the Azure SDK for Java | Microsoft Docs
description: Learn how to download the Azure SDK for Java, with sample code provided for Maven projects.
services: ''
documentationcenter: java
author: routlaw
manager: douge
editor: ''

ms.assetid: 4b8f8fe6-1b26-4bb4-9be9-6ae757a59e66
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: multiple
ms.devlang: Java
ms.topic: article
ms.date: 12/22/2016
ms.author: routlaw;asirveda

---
# Download the Azure SDK for Java

Use the libraries in the Java SDK for Azure to manage and use Azure services and resources in your applications.  
For an overview of Azure and Java, visit the [Java developer center for Azure](https://azure.microsoft.com/en-us/develop/java)

## Installation

### Maven

Add a dependency element in your pom.xml to add a library from the SDK to your [Maven](https://maven.apache.org) project.  

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure</artifactId>
    <version>n.n.n</version>
</dependency>
```

### Gradle

Add a compile entry in the dependency section of your build.gradle file to add a library from the SDK to your [Gradle](https://gradle.org) project.

```json
dependencies {
    compile 'com.microsoft.azure:azure:n.n.n'
}
```

### Manual download

Select the JAR link next to the library from the Java SDK for Azure to download the latest release of the library. Use one of the following options to add the JAR files to your project:

* Import the JAR files into your Java project in Eclipse or IntelliJ.
* Configure the build paths for your Java projects in Eclipse or IntelliJ to include the path to the JAR files.

## Azure services

Key Vault - Securely store secrets and keys.
SQL Database - Set up and configure a SQL database.
DocumentDB - Create NoSQL databases with JSON data.
Redis Cache - High throughput, in-memory key/value cache for your apps.
Storage - Store, read, and update data to Azure Storage services.
Storage for Android - Access Azure Storage services from your Android applications.
Event Hub - Send, receive, and process telemetry events in your application.
Data Lake Store - Store and analyze all your data in a single location with no constraints.
IoT Service - Create and manage your IoT hubs in Azure
IoT Device - Send data to an IoT hub from your device.
AppInsights - Gather logs and telemetry to track performance and usage of your app.
Active Directory Authentication - Enable secure sign-in and authorization.



## Azure resource management
Content Delivery Network - Provide lower latency and high bandwidth for content no matter where your users are.


## Azure service management



## See Also
For more information about the Azure Toolkits for Java IDEs, see the following links:

* [Azure Toolkit for Eclipse]
  * [Installing the Azure Toolkit for Eclipse]
  * [Create a Hello World Web App for Azure in Eclipse]
  * [What's New in the Azure Toolkit for Eclipse]
* [Azure Toolkit for IntelliJ]
  * [Installing the Azure Toolkit for IntelliJ]
  * [Create a Hello World Web App for Azure in IntelliJ]
  * [What's New in the Azure Toolkit for IntelliJ]

For more information about using Azure with Java, see the [Azure Java Developer Center].

> [!NOTE]
> For detailed information on setting up build paths in Eclipse, see the [Java Build Path] article at the Eclipse website.
>

<!-- URL List -->

[Azure Toolkit for Eclipse]: ./azure-toolkit-for-eclipse.md
[Azure Toolkit for IntelliJ]: ./azure-toolkit-for-intellij.md
[Create a Hello World Web App for Azure in Eclipse]: ./app-service-web/app-service-web-eclipse-create-hello-world-web-app.md
[Create a Hello World Web App for Azure in IntelliJ]: ./app-service-web/app-service-web-intellij-create-hello-world-web-app.md
[Installing the Azure Toolkit for Eclipse]: ./azure-toolkit-for-eclipse-installation.md
[Installing the Azure Toolkit for IntelliJ]: ./azure-toolkit-for-intellij-installation.md
[What's New in the Azure Toolkit for Eclipse]: ./azure-toolkit-for-eclipse-whats-new.md
[What's New in the Azure Toolkit for IntelliJ]: ./azure-toolkit-for-intellij-whats-new.md

[Azure Java Developer Center]: http://go.microsoft.com/fwlink/?LinkID=699547
[Azure Libraries Repository on Maven]: http://go.microsoft.com/fwlink/?LinkID=286274
[Java Build Path]: http://help.eclipse.org/luna/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2Freference%2Fref-properties-build-path.htm
[license]: http://www.apache.org/licenses/LICENSE-2.0.html
[maven-getting-started]: http://go.microsoft.com/fwlink/?LinkID=622998