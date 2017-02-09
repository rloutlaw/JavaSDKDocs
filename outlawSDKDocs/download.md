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
   
For an overview of Azure and Java, visit the [Java developer center for Azure](https://azure.microsoft.com/en-us/develop/java).

## Installation

### Maven

Add a dependency entry in your pom.xml to add a library from the SDK to your [Maven](https://maven.apache.org) project.

For example, to include the latest version of the Azure Management SDK for Java:

```XML
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure</artifactId>
    <version>1.0.0-beta5</version>
</dependency>
``` 
### Gradle

Create an entry in the dependency section of your build.gradle file to add a library from the SDK to your [Gradle](https://gradle.org) project.

```
dependencies {
    compile 'com.microsoft.azure:azure:1.0.0-beta5'
}
```

## Azure services

[Key Vault](https://docs.microsoft.com/en-us/azure/key-vault) - Encrypt secrets and keys and safely access them from your applications. 

```XML
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-keyvault</artifactId>
    <version>1.0.0-beta3</version>
</dependency>
```

[Reference](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.keyvault) | [Samples](https://github.com/Azure-Samples/batch-keyvault-java-management) | [GitHub](https://github.com/Azure/azure-sdk-for-java)  

DocumentDB - Create NoSQL databases with JSON documents and query them with SQL or JavaScript syntax.   

```XML
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-documentdb</artifactId>
    <version>1.9.3</version>
</dependency>
```

[Reference](http://azure.github.io/azure-documentdb-java/) | [Samples](https://docs.microsoft.com/en-us/azure/documentdb/documentdb-java-application) | [GitHub](https://github.com/Azure/azure-documentdb-java)   

Redis Cache - High throughput, in-memory key/value cache for your apps. Maven | Gradle | Download | Reference | Samples  
Storage - Store, read, and update data to Azure Storage services. Maven | Gradle | Download | Reference | Samples  
Storage for Android - Access Azure Storage services from your Android applications.  
Event Hub - Send, receive, and process telemetry events in your application.  
Data Lake Store - Store and analyze all your data in a single location with no constraints.  
IoT Service - Create and manage your IoT hubs in Azure   
IoT Device - Send data to an IoT hub from your device.  
AppInsights - Gather logs and telemetry to track performance and usage of your app.  
Active Directory Authentication - Enable secure sign-in and authorization.  


## Azure resource management
SQL Database - Set up and configure a SQL database.  

Reference | Samples | GitHub   

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