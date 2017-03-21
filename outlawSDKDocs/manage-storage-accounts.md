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

# Manage Azure storage accounts with Java

[This sample](https://github.com/Azure-Samples/storage-java-manage-storage-accounts) creates an [Azure Storage](https://docs.microsoft.com/en-us/azure/storage/storage-introduction) account and works with the account access keys using the [Java management libraries](https://github.com/Azure/azure-sdk-for-java). 

## Sample code 

[View the complete code sample on GitHub](https://github.com/Azure-Samples/storage-java-manage-storage-accounts).

### Authenticate with Azure

[!INCLUDE [auth-include](_shared/auth-include.md)]

### Create a storage account

```java
// create a new storage account
StorageAccount storageAccount = azure.storageAccounts().define(storageAccountName)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup(rgName)
                    .create();
```

### List keys in a storage account
```java
// list the name and value for each access key in the storage account
List<StorageAccountKey> storageAccountKeys = storageAccount.getKeys();
for(StorageAccountKey key : storageAccountKeys)    {
    System.out.println("Key name: " + key.keyName() + " with value "+ key.value());
}
```

### Regenerate a key in a storage account

```java
// regenerate the first key in a storage account and return an updated list of keys 
List<StorageAccountKey> updatedStorageAccountKeys =
    storageAccount.regenerateKey(storageAccountKeys.get(0).keyName());
```

### List all storage accounts in a resource group
```java
// get a list of accounts in a resource group , log info about each one
List<StorageAccount> accounts = azure.storageAccounts().listByGroup(rgName);
for (StorageAccount sa : accounts) {
    System.out.println("Storage Account " + sa.name() + " created @ " + sa.creationTime());
}
```

### Delete a storage account
```java
// delete by ID when you already have a storage account object
azure.storageAccounts().deleteById(storageAccount.id());

// delete by resource group and account name if you don't have an account object
azure.storageAccounts().deleteByGroup(rgName,accountName);
```

## Sample explanation

[The sample code on GitHub](https://github.com/Azure-Samples/storage-java-manage-storage-accounts) creates a storage account, reads and updates access keys, lists all storage accounts in a resource group, and then deletes the storage account before exiting

| Class used in sample | Notes
|-------|-------|
| [com.microsoft.azure.management.storage.StorageAccounts](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.datalake.analytics._storage_accounts) | Created from the `azure.storageAccounts()` entry point. Provides create, list, update, and delete operations for storage accounts.
| [com.microsoft.azure.management.storage.StorageAccount](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.storage._storage_account)  | Representation of an Azure storage account. Use the methods in the class to get information about the storage account.
| [com.microsoft.azure.management.storage.StorageAccountKey](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.storage._storage_account_key) | `StorageAccount.getKeys()` returns a list of these objects. Use the `regenerateKey` methods in `StorageAccount` to update the keys.



## Next steps

[!INCLUDE [next-steps](_shared/next-steps.md)]

Additional Azure storage samples can be found in the [Azure storage documentation](https://docs.microsoft.com/en-us/azure/storage/).