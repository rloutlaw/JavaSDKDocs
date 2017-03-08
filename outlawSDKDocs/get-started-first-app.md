---
title: Deploy an app to Azure using the Java API | Microsoft Docs
description: Deploy a Java web application to Azure from your Java code
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
# Get started with Azure development with Java and Eclipse

This guide walks you through setting up your command line environment for Azure development and deploying a Java web application in [Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/).

## Prerequisites

Before starting, make sure you have installed and configured the following:

- [Java 8 JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [Apache Maven](https://maven.apache.org)
- [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2)
- [Git](https://git-scm.org)

## Download the sample

[Clone the sample code from GitHub](https://github.com/rloutlaw/hello-world-java) or [download and extract the sample](sample.zip) to a folder on your system.

## Update the Maven project 

In the folder with the sample code, open up the `pom.xml` in the sample and make the following changes:

  <groupId>com.<username>.azure.appdemo</groupId>
  <artifactId>AzureAppDemo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>AzureAppDemo</name>

## Test the application locally

Build the sample webapp and test it on localhost using the Jetty plugin:

```bash
mvn clean package
mvn jetty:run
```

Open your browser to http://localhost:8080 . You should see the current time displayed. Use control+C in the command console to stop the Jetty server.

## Create your App Service plan for the app and create 

From the Azure CLI 2.0, run the following to create a plan in Azure App Service's free tier, create an application in that plan , and update the application's config to use Tomcat.

```bash
appname = AzureAppDemo$RANDOM
az group create -n sampleResourceGroup -l westus2 
az appservice plan create --name $appname --resource-group sampleResourceGroup --sku FREE
az appservice web create --name $webappname --resource-group sampleResourceGroup --plan $webappname
az appservice web config update --resource-group sampleResourceGroup --name $webappname --java-container TOMCAT --java-version 1.8.0_73 --java-container-version 8.5
```

## Push the application using Git

## Test the application

## Make some changes

## Verify the changes

## Learn more

See the How-Tos for more detailed samples on how to integrate Azure services into your application and manage your Azure resources from your code.
