---
title: Manage Azure storage accounts with Java | Microsoft Docs
description: Sample code to manage Azure storage accounts in your Java code
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

# Manage Azure storage networks in Java

Manage [Azure Storage](https://docs.microsoft.com/en-us/azure/storage/storage-introduction) accounts and access keys using the Java management libraries. [The full sample on GitHub](https://github.com/Azure-Samples/storage-java-manage-storage-accounts) creates a storage account, reads and updates access keys, lists all storage accounts in a resource group, and deletes a storage account.

## Authenticate with Azure

Create an [authentication file](https://github.com/Azure/azure-sdk-for-java/blob/master/AUTH.md) and set the environment variable `AZURE_AUTH_LOCATION` on the command line with the full path to the file.

```bash
export AZURE_AUTH_LOCATION=/Users/raisa/azure.auth
```

The authentication file is used to create the entry point `Azure` object used by the management libraries to define, create, and configure Azure resources.

```java
// pull in the location of the security file from the environment 
final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));

// generate the entry point Azure object
Azure azure = Azure
        .configure()
        .withLogLevel(LogLevel.NONE)
        .authenticate(credFile)
        .withDefaultSubscription();
```

## Create a storage account

```java
// generate a new storage account using a randomly generated account name
final String storageAccountName = Utils.createRandomName("sa");
StorageAccount storageAccount = azure.storageAccounts().define(storageAccountName)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup(rgName)
                    .create();
```

## List keys in a storage account

```java
            // list the name and value for each key in the storage account
            List<StorageAccountKey> storageAccountKeys = storageAccount.getKeys();
            for(StorageAccountKey key : storageAccountKeys)    {
                System.out.println("Key name: " + key.keyName() + " with value "+ key.value());
            }
```

## Regenerate a key in a storage account

```java
// regenerate the first key in a storage account, returning an updated list of keys to work with
List<StorageAccountKey> updatedStorageAccountKeys = storageAccount.regenerateKey(storageAccountKeys.get(0).keyName());
```

## List storage accounts in a resource group

```java
            List<StorageAccount> accounts = azure.storageAccounts().listByGroup(rgName);
            for (StorageAccount sa : accounts) {
                System.out.println("Storage Account " + sa.name()
                        + " created @ " + sa.creationTime());
            }
```

## Delete a storage account

```java
            // delete by ID when you already have a storage account object
            System.out.println("Deleting storage account - " + storageAccount.name()
                    + " created @ " + storageAccount.creationTime());

            azure.storageAccounts().deleteById(storageAccount.id());
```

