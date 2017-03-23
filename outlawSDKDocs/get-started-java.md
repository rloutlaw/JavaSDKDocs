---
title: Get started with Java in Azure | Microsoft Docs
description: Set up your account and verify prerequistes for the demo
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK
author: routlaw
manager: douge
ms.assetid: 5e0e1572-2bd4-4bb5-88a6-43f383f9c240
ms.service: Azure
ms.devlang: java
ms.topic: get-started-article
ms.technology: Azure
ms.date: 3/06/2016
---

# Get started with Java for Azure developers

This tutorial will walk you through deploying a simple Java webapp in Azure.

## Prerequisites

- An Azure account. If you don't have one , [get a free trial](https://azure.microsoft.com/free/)
- [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [Maven 3](http://maven.apache.org/download.cgi)
- [Git](https://git-scm.com/downloads)
- [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2)

## Get the sample

Clone the [sample Maven project](https://github.com/rloutlaw/hello-world-java) to your system:

```bash
git clone https://github.com/rloutlaw/hello-world-java.git
```

## Run the sample

Build the sample and run it in a local [Jetty](http://www.eclipse.org/jetty/) container:

```bash
cd hello-world-java
mvn package jetty:run
```

Browse to http://localhost:8080/index.jsp to verify that the app is running.

## Configure App Service

Set up a Tomcat environment in Azure using the Azure CLI 2.0 to run the sample in.

```bash
# create all resources under the same resource group
az group create -n sampleResourceGroup -l westus2

# create a default webapp and resource group for future commands
appname=AzureJavaDemo$RANDOM
az configure --defaults web=$appname group=sampleResourceGroup

# create the webapp
az appservice web create --name $appname

# configure the webapp to use Tomcat 
az appservice web config update --java-container TOMCAT --java-version 1.8.0_73 --java-container-version 8.5
```

## Deploy the sample 

Deploy the sample to Azure App Service. 

```bash
# export the FTP URL and credentials for the webapp to bash
read AZSITE AZUSER AZPASS <<< $(az appservice web deployment list-publishing-profiles --query "[?publishMethod=='FTP'].{URL:publishUrl, Username:userName,Password:userPWD}" --output tsv)
export AZSITE AZUSER AZPASS

# deploy using Maven reading in the FTP information from the shell
mvn install -s az-settings.xml
```

## Verify in your browser

```bash
az appservice web browse
```

