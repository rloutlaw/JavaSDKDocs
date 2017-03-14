---
title: Get started with Java in Azure | Microsoft Docs
description: Add storage to your Java web app running in Azure App Service
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK
author: routlaw
manager: douge
ms.assetid: f63d6980-7ce1-4ab0-9b14-ffe34733c669
ms.service: Azure
ms.devlang: java
ms.topic: reference
ms.technology: Azure
ms.date: 3/06/2016
---

# Get started with Java in Azure

## Upload and store a image in Azure storage

Add functionality to save to and display images from [Azure blob storage](https://docs.microsoft.com/en-us/azure/storage/storage-java-how-to-use-blob-storage) in the sample.

## Create a storage account and add the connection string to the web app via environment variable

```bash
storage_acct="helloworld$RANDOM"
az storage account create --name $storage_acct --resourceGroup sampleResourceGroup --location westus2
connstr=$(az storage account show-connection-string --name $storageName --resource-group sampleResourceGroup --query connectionString --output tsv)
az appservice web config appsettings update --settings "STORAGE_CONNSTR=$connstr" --name $appname --resource-group sampleResourceGroup
az storage container create  --connection-string $connstr --name helloworld 
az storage container set-permission --connection-string $connstr --name helloworld --public-access container
```

## Add a form to the JSP to upload an image

```html
<h2>Upload a file</h2><br/>
<form method="post" action="upload" enctype="multipart/form-data">
<input type="file" name="file"/><br/>
 <input type="submit" value="Upload to Azure Storage" name="upload"></h3></form>
```

The servlet called to upload the file is [included](https://github.com/rloutlaw/hello-world-java/src/main/java/com/microsoft/azure/samples/AzureStorageUploadServlet.java) in the sample. 

## Display the image in the storage account 

Add an image tag to the JSP, replacing `$storage_acct` with the actual name of the storage account generated:

```html
<img src="https://<$storage_acct>.blob.core.windows.net/helloworld/helloworld.jpg">
```

>[!div class="step-by-step"]
[**Automate deployment** &rarr;](get-started-automate.md)
