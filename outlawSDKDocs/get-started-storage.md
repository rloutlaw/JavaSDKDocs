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

Add functionality to store and display images in [Azure storage](https://docs.microsoft.com/en-us/azure/storage/storage-java-how-to-use-blob-storage) to the sample.

## Create and configure a storage account 

```bash
storageName=helloworld$RANDOM
az storage account create --name $storageName --resource-group sampleResourceGroup \
 --location $location --sku Standard_LRS
```

This creates the Azure storage account, but the Java application needs to interact with a [blob storage]((https://docs.microsoft.com/en-us/azure/storage/storage-java-how-to-use-blob-storage)) container created in the account.

## Add a connection string to the webapp and create the storage container

```bash
connstr=$(az storage account show-connection-string --name $storageName --resource-group sampleResourceGroup --query connectionString --output tsv)
az appservice web config appsettings update --settings "STORAGE_CONNSTR=$connstr" --name $appname --resource-group sampleResourceGroup
az storage container create --connection-string $connstr --name helloworld --public-access container
```

Create the `helloworld` container in the storage account to store and serve an uploaded image file.

## Create a form on the JSP to upload an image

```html
<h2>Upload a file</h2><br/>
<form method="post" action="upload" enctype="multipart/form-data">
    <input type="file" name="file"/><br/>
    <input type="submit" value="Upload to Azure Storage" name="upload">
</form>
```

This form calls an [upload servlet](https://github.com/rloutlaw/hello-world-java/src/main/java/com/microsoft/azure/samples/AzureStorageUploadServlet.java) to store a local file to the blob storage container.

## Display the image in the storage account 

Add an image tag to the JSP, replacing `storageName` with the name of the storage account in the previous steps:

```html
<img src="https://<$storageName>.blob.core.windows.net/helloworld/helloworld.jpg">
```

## Rebuild deploy the updated sample
```
mvn clean package install -s az-settings.xml
```

Refreshing the web browser will show the new form and a broken image tag. Upload an image from your computer using the form and refresh your browser to display the image from Azure storage.