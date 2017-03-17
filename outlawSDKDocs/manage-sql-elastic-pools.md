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

[This sample](https://github.com/Azure-Samples/sql-database-java-manage-sql-dbs-in-elastic-pool) creates a SQL database server with an [elastic pool](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-elastic-pool) to manage and scale multiple databases. 

## Authenticate with Azure

[!INCLUDE [auth-include](_shared/auth-include.md)]

## Create a SQL database instance with an elastic pool

```java
SqlServer sqlServer = azure.sqlServers().define(sqlServerName)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup(rgName)
                    .withAdministratorLogin(administratorLogin)
                    .withAdministratorPassword(administratorPassword)
                    // use ElasticPoolEditions.STANDARD as the edition and creating two databases
                    .withNewElasticPool(elasticPoolName, ElasticPoolEditions.STANDARD, database1Name, database2Name)
                    .create();
```

See the [ElasticPoolEditions class reference](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._elastic_pool_editions) for current edition values. Review the [SQL database elastic pool documentation](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-elastic-pool) to compare edition limits and characteristics. 

## Sample code

### Change Database Transaction Unit (DTU) settings in an elastic pool

```java
// set an pool of 200 eDTUs to share between the databases
// and ensure that a database always has 10 DTUs available but never uses more than 50
elasticPool = elasticPool.update()
                    .withDtu(200)
                    .withDatabaseDtuMin(10)
                    .withDatabaseDtuMax(50)
                    .apply();
```

Review the [DTUs and eDTUs documentation](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-what-is-a-dtu) to learn more about allocating resources in a service tier to multiple databases.

### Create a new database and add it to an elastic pool

```java
SqlDatabase anotherDatabase = sqlServer.databases().define(anotherDatabaseName).create();
// update the SqlElasticPool object with the newly created SqlDatabase
elasticPool.update().withExistingDatabase(anotherDatabase).apply();            
```

### Remove a database from an elastic pool
```java
// assign the database an edition since its resources are no longer allocated by the pool 
            anotherDatabase = anotherDatabase.update()
                    .withoutElasticPool()
                    .withEdition(DatabaseEditions.STANDARD)
                    .apply();
```

See the [DatabaseEditions class reference](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._database_editions) for the current field values to pass to `withEdition`.

### List current activities in an elastic pool
```java
// get a list of in-flight elastic operations in the pool and log them 
for (ElasticPoolDatabaseActivity databaseActivity : elasticPool.listDatabaseActivities()) {
    System.out.println("Database " + databaseActivity.databaseName() + " performing operation " 
        + databaseActivity.operation() + " with objective " + databaseActivity.requestedServiceObjective());
}
```

### List databases in an elastic pool
```java
// write a list of databases in the elastic pool to the console
for (SqlDatabase databaseInServer : elasticPool.listDatabases()) {
    System.out.println(databaseInServer.name());
}
```

### Delete an elastic pool
```java
sqlServer.elasticPools().delete(elasticPoolName);
```

## Sample explanation

The sample uses the following classes in the [Azure management libary] to create and work with SQL server instances, elastic pools, and SQL databases.

| Class | Notes |
| [com.microsoft.azure.management.sql.SqlServer](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._sql_server) | SQL Server instance in Azure created by `azure.sqlServers().define()...create() fluent chain. Provides methods to create elastic pools and databases in the created instance. |
| [com.microsoft.azure.management.sql.SqlDatabase](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._sql_database) | Client side object representing a SQL database. Instances created through 'sqlServer().define()...create()` are created in Azure and their representation returned from the `create()`. | 
| [com.microsoft.azure.management.sql.DatabaseEditions](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._database_editions) | Constant static fields used to set database resources when creating a database outside of an elastic pool or when moving a database out of an elastic pool  | 
| [com.microsoft.azure.management.sql.SqlElasticPool] | | 
| [com.microsoft.azure.management.sql.ElasticPoolEditions] | | 
| [com.microsoft.azure.management.sql.ElasticPoolDatabaseActivity] | | 
| [com.microsoft.azure.management.sql.ElasticPoolActivity] | | 

