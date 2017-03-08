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

[Clone the sample code from GitHub](https://test) or [download and extract the sample](sample.war) to a folder on your system.

## Update the Maven project 

In the folder with the sample code, open up the `pom.xml` in the sample and make the following changes:

  <groupId>com.<username>.azure.appdemo</groupId>
  <artifactId>AzureAppDemo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>AzureAppDemo</name>

## Test the application locally

## Import Azure dependencies

Run `mvn package` to import the dependencies from Maven Central and build the sample.

## Create your App Service plan for the app and create 

From the Azure CLI 2.0, run the following to create a plan in Azure App Service's free tier, create an application in that plan , and update the application's config to use Tomcat.

```bash
appname = AzureAppDemo$RANDOM
az group create -n sampleResourceGroup -l westus2 
az appservice plan create --name $appname --resource-group sampleResourceGroup --sku FREE
az appservice web create --name $webappname --resource-group sampleResourceGroup --plan $webappname
az appservice web config update --resource-group sampleResourceGroup --name $webappname --java-container TOMCAT --java-version 1.8.0_73 --java-container-version 8.5
```



## Upload the app

## Test the application

## Make some changes

## Verify the changes

## Set up authentication

Register your application with Azure Active Directory so your app can create and update resources in Azure. This guide uses the [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2) , but you can also [create the service principal via the web portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal).

### Create a service principal

Create a [service principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-application-objects#application-registration) to authenticate your application.
Service principals let you set application access separate from user accounts and identities. 

1. Log in using the Azure CLI 2.0 with `az login`. 
2. List the subscriptions for your account with `az account list`.
3. Select the subscription for your service principal to access with `az account set --subscription <subscription name>`. 
4. Create the service principal with `az ad sp create-for-rbac -n "AzureMgmtDemo" --role reader --output json`. Keep the output from this command to a safe place as it has information you'll
need to use in the next step.

### Create the credential file

Create a properties file with the service principal information from the previous step. This file will be passed to the Azure management API to authenticate our app.

Right-click the **src/main/resources** folder in **Project Explorer** and select **New** > **File**. Enter *auth.propeties* in the **File name:** field, then select **OK**.   
   ![Create the auth properties file in the regular folders, not the source folders, in Eclipse](_img/eclipse_auth_location.png)   

Enter the following properties into this file:

```
subscription=########-####-####-####-############
client=########-####-####-####-############
key=XXXXXXXXXXXXXXXX
tenant=########-####-####-####-############
managementURI=https\://management.core.windows.net/
baseURL=https\://management.azure.com/
authURL=https\://login.windows.net/
graphURL=https\://graph.windows.net/
```

Update this property file with the following changes:

- subscription = use the *id* value from `az account list`
- client = use the *appId* value from the output taken from the service principal created in the previous step
- key = use the *password* value from the service principal creation output
- tenant - use the *tenant* value from the service principal creation output

## Create the application

Create a new class in the **src/main/java** source folder Eclipse (Right click the source folder, then select **New** > **Class**). Enter AzureAppDemo in the **Name:** field. 
Keep the defaults for the rest of the options and select **Finish**. 

Eclipse will open the source file for editing. Enter the following code into the file:

```java
import com.microsoft.azure.management.compute.VirtualMachine;
import com.microsoft.azure.management.Azure;
import com.microsoft.azure.PagedList;

import java.lang.ClassLoader;
import java.io.File;

public class AzureAppDemo {

    public static void main(String[] args) {

        try {

            // authenticate the app with Azure using the properties file created earlier
            ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
            File authfile = new File(classLoader.getResource("auth.properties").getFile());
            Azure azure = Azure.authenticate(authfile).withDefaultSubscription();

			// get a list of virtual machines
            PagedList<VirtualMachine> vmlist = azure.virtualMachines().list();
            for (VirtualMachine vm : vmlist) {
                if (vm != null) {
                    // write information about the VMs to the console
                    System.out.println("Found virtual machine " + vm.name()
                            + " with size " + vm.size() + " in resource group " + vm.resourceGroupName()
                            + " with state " + vm.powerState());
                }
            }
        } catch (Exception e) {
            System.err.println("Error listing virtual machines in resource groups: "
                    + e.getMessage());
        }
    }
}
```

## Run the sample 

Run the code by selecting the Run button in Eclipse ( ![run button in Eclipse](_img/eclipse_run_button.png) , shortcut Ctrl+F11) or from the **Run** menu. The output from the application will display in the **Console** window.

```
Found virtual machine myAzureVM with size Standard_DS1_v2 in resource group myazresgroup with state PowerState/running
```

## Learn more

See the How-Tos for more detailed samples on how to integrate Azure services into your application and manage your Azure resources from your code.
