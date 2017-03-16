---
title: Manage Azure SQL databases in elastic pools with Java | Microsoft Docs
description: Sample code to manage Azure SQL databases using elastic pools in your Java code
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

# Manage Azure SQL database elastic pools with Java

[This sample](https://github.com/Azure-Samples/sql-database-java-manage-sql-dbs-in-elastic-pool) 

## Authenticate with Azure

[!INCLUDE [auth-include](_shared/auth-include.md)]

## Create a SQL database instance with an elastic pool

```java
SqlServer sqlServer = azure.sqlServers().define(sqlServerName)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup(rgName)
                    .withAdministratorLogin(administratorLogin)
                    .withAdministratorPassword(administratorPassword)
                    // using ElasticPoolEditions.STANDARD as the edition and creating two databases
                    .withNewElasticPool(elasticPoolName, elasticPoolEdition, database1Name, database2Name)
                    .create();
```