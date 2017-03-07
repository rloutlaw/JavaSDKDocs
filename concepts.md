---
title: Azure management library concepts | Microsoft Docs
description: Understand patterns and concepts used by the Azure management libraries for Java
services: ''
documentationcenter: java
author: routlaw
manager: douge
editor: ''

ms.assetid: 9b461de8-46bc-4650-8e9e-59531f4e2a53
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: multiple
ms.devlang: Java
ms.topic: article
ms.date: 12/22/2016
ms.author: routlaw;asirveda

---

# Azure management library concepts

## Authentication

The simplest authentication scenario when using the Azure management libraries is to create an external properties file that contains authentication details and use it to build the top-level Azure object:

```java
// pull in the location of the authenticaiton properties file from the environment 
final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));

Azure azure = Azure
        .configure()
        .withLogLevel(LogLevel.NONE)
        .authenticate(credFile)
        .withDefaultSubscription();
```

This pattern is used in the samples since the code is compact and simple to follow.

This requires being comfortable with putting credentials on the filesystem and making it readable by your Java code. For more detailed scenarios including secure secret management using Azure Key Vault, see the [complete authentication article](authentication.md).

## Creatables and Updateables / in-memory computation vs. calling the Azure backend

One of the challenges when working with resources is that you need other Azure resources to define a new resource, such as a IP address for a virtual machine or a firewall definition for a SQL database. 
You don't want to have to create every single resource, wait for it to be created in Azure,  and verify it exists before moving onto the next item to create. Creating each object asynchronously is challenging because some resources can be safely created in parallel with other and some need to have the resources created beforehand, and it's not easy to know which is the case for your scenario. Concurrent code like this also can be difficult to develop, debug, maintain and extend later.

The Azure libraries for Java provide a mechanism to locally define Azure resources and use those definitions when creating and updating other Azure resource using Creatable objects. Creatables are generated through the resource's `define()` method, for example:

```java
Creatable<PublicIPAddress> publicIPAddress2 = azure.publicIPAddresses().define(publicIPAddressName2)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup(rgName);
```


## Child resources

## Lists and iterations

## Paralell creation of resources

## Reactive patterns and Observables

## Builders, not constructors

## Actionable verbs

## Exception handling

## Returned object collections

## Logs and trace
