---
title: Get started with Java in Azure | Microsoft Docs
description: Add storage to your Java web app running in Azure App Service
keywords: Azure Java, Azure Java API Reference, Azure Java class library, Azure SDK
author: routlaw
manager: douge
ms.assetid: 7b92e776-959b-4632-8b1d-047ce1417616
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
az appservice web config appsettings update --settings "STORAGE_CONNECTION=$connstr" --name $appname --resource-group sampleResourceGroup
```

## Add a form to upload an image to the JSP

```html
<h2>Upload a file</h2>
Select a file to upload: <br/>
<form action="UploadAzureStorageServlet" method="post" enctype="multipart/form-data">
  <input type="file" name="file" size"40" />
  <br/>
  <input type="submit" value="Upload" />
</form>
```

The servlet called to upload the file is already included in the sample. The relevant code from it is here:

## Display the helloworld image saved to the storage account under the timestamp

>[!div class="step-by-step"]
[**Automate deployment* &rarr;](get-started-automate.md)
