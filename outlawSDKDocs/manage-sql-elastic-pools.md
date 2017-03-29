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

[This sample](https://github.com/Azure-Samples/sql-database-java-manage-sql-dbs-in-elastic-pool) creates a SQL database server with an [elastic pool](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-elastic-pool) to manage and scale resouces in a single SQL database plan across multiple databases. 

## Sample code

Create an [authentication file](https://github.com/Azure/azure-sdk-for-java/blob/master/AUTH.md) and set an environment variable `AZURE_AUTH_LOCATION` with the full path to the file on your computer. Then run:

```
git clone https://github.com/Azure-Samples/sql-database-java-manage-sql-dbs-in-elastic-pool
cd sql-database-java-manage-sql-dbs-in-elastic-pool
mvn clean compile exec:java
```

[View the complete code sample on GitHub](https://github.com/Azure-Samples/sql-database-java-manage-sql-dbs-in-elastic-pool)

## Authenticate with Azure

[!INCLUDE [auth-include](_shared/auth-include.md)]

## Create a  SQL database logical server with an elastic pool

```java
SqlServer sqlServer = azure.sqlServers().define(sqlServerName)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup(rgName)
                    .withAdministratorLogin(administratorLogin)
                    .withAdministratorPassword(administratorPassword)
                    // use ElasticPoolEditions.STANDARD as the edition
                    // and creating two databases
                    .withNewElasticPool(elasticPoolName, ElasticPoolEditions.STANDARD, 
                        database1Name, database2Name)
                    .create();
```

See the [ElasticPoolEditions class reference](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._elastic_pool_editions) for current edition values. Review the [SQL database elastic pool documentation](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-elastic-pool) to compare edition resource characteristics. 

## Change Database Transaction Unit (DTU) settings in an elastic pool

```java
// set an pool of 200 eDTUs to share between the databases
// and ensure that a database always has 10 DTUs available but never uses more than 50
elasticPool = elasticPool.update()
                    .withDtu(200)
                    .withDatabaseDtuMin(10)
                    .withDatabaseDtuMax(50)
                    .apply();
```

Review the [DTUs and eDTUs documentation](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-what-is-a-dtu) to learn about allocating resources in a service tier to multiple databases.

## Create a new database and add it to an elastic pool

```java
// update the elasticPool object with the newly created SqlDatabase
SqlDatabase anotherDatabase = sqlServer.databases().define(anotherDatabaseName).create();
elasticPool.update().withExistingDatabase(anotherDatabase).apply();            
```

The API created `anotherDatabase` at [S0 tier](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-service-tiers) in the first statement. Moving `anotherDatabase` to the elastic pool assigns the database resources based on the pool settings.

## Remove a database from an elastic pool
```java
// assign an edition since the database resources are no longer managed in the pool 
anotherDatabase = anotherDatabase.update()
                     .withoutElasticPool()
                     .withEdition(DatabaseEditions.STANDARD)
                     .apply();
```

See the [DatabaseEditions class reference](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._database_editions) for values to pass to `withEdition()`.

## List current database activities in an elastic pool
```java
// get a list of in-flight elastic operations on databases in the pool and log them 
for (ElasticPoolDatabaseActivity databaseActivity : elasticPool.listDatabaseActivities()) {
    System.out.println("Database " + databaseActivity.databaseName() 
        + " performing operation " + databaseActivity.operation() + 
        " with objective " + databaseActivity.requestedServiceObjective());
}
```

Database activities include moving an existing database in and out of an elastic pool or the creation or deletion of a database already in an elastic pool.


## List databases in an elastic pool
```java
// log a list of databases in the elastic pool to the console
for (SqlDatabase databaseInServer : elasticPool.listDatabases()) {
    System.out.println(databaseInServer.name());
}
```

Review the methods in [com.microsoft.azure.management.sql.SqlDatabase](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._sql_database) to query the database objects in more detail when iterating over them.

## Delete an elastic pool
```java
sqlServer.elasticPools().delete(elasticPoolName);
```

Remove a database from an elastic pool to assign fixed resources to it defined by its [pricing tier](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-service-tiers).

## Sample code summary.

The sample creates a SQL server instance with two databases managed in a single elasic pool. The elastic pool resource limits are updated, and additional databases are added to the pool. The elastic pool is then queried for its configuration and state. 

The sample deletes all resources it created before exiting.

| Class used in sample | Notes |
|-------|-------|
| [com.microsoft.azure.management.sql.SqlServer](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._sql_server) | SQL Server instance in Azure created by `azure.sqlServers().define()...create()` fluent chain. Provides methods to create elastic pools and databases in the created instance. 
| [com.microsoft.azure.management.sql.SqlDatabase](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._sql_database) | Client side object representing a SQL database. Created through `sqlServer().define()...create()`. 
| [com.microsoft.azure.management.sql.DatabaseEditions](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._database_editions) | Constant static fields used to set database resources when creating a database outside of an elastic pool or when moving a database out of an elastic pool  
| [com.microsoft.azure.management.sql.SqlElasticPool](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._sql_elastic_pool) | Created from the `withNewElasticPool()` section of the fluent chain that created the SqlServer in Azure. Provides methods to set resource limits for databases running in the elastic pool and for the elastic pool itself. 
| [com.microsoft.azure.management.sql.ElasticPoolEditions](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._elastic_pool_editions) | Class of constant fields defining the resources available to an elastic pool. See [SQL database elastic pool documentation](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-elastic-pool) for tier details. 
| [com.microsoft.azure.management.sql.ElasticPoolDatabaseActivity](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._elastic_pool_database_activity) | Retreived from `SqlElasticPool.listDatabaseActivities()`. Each object of this type represents an activity performed on a database in the elastic pool.
| [com.microsoft.azure.management.sql.ElasticPoolActivity](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.sql._elastic_pool_activity) | Retrieved in a List from `SqlElasticPool.listActivities()`. Each of object in the list represents an activity performed on the elastic pool (not the databases in the elastic pool).

## Next steps

[!INCLUDE [next-steps](_shared/next-steps.md)]