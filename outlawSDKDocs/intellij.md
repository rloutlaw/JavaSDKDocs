---
title: Get started with Azure development in IntelliJ | Microsoft Docs
description: Learn how to start working with the Azure and Java in the IntelliJ IDEA IDE.
services: ''
documentationcenter: java
author: routlaw
manager: douge
editor: ''

ms.assetid: 33487ba9-5622-4fb0-95ee-ed851f94d9ae
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: multiple
ms.devlang: Java
ms.topic: article
ms.date: 02/13/2016
ms.author: routlaw

---
# Get started with Azure development with Java and IntelliJ 

This guide walks you through creating a small command line tool with IntelliJ that lists your virtual machines in Azure.

## Prerequisites

- [IntelliJ IDEA](https://www.jetbrains.com/idea)
- [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2)

## Create your project

1. Open IntelliJ and select **Create New Project**. Select **Maven** from the list on the left, then select **Next** (do not select **Create from archetype**).
    ![Complete the configure project step in the New Maven project dialog](_img/create_maven_project_intellij.png)
2. On the second page of **New Project**, use the following values:

   - Groupid: `com.<username>.azure.mgmtdemo`  
   - Artifactid: AzureMgmtDemo  
   - Version: 0.0.1-SNAPSHOT  

   Select **Next**.
3. Use *AzureMgmtDemo* for the **Project name** and point the **Project location** to whatever value you like. Select **Finish**. 

## Import the Maven dependencies

Double-click the **pom.xml** file in the **Project** view.   

Add the following XML before the `</project>` tag.

```XML
<dependencies>
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure</artifactId>
    <version>1.0.0-beta5</version>
</dependency>
</dependencies>
```

Save the changes. If you don't have auto-import of Maven dependencies set up, you'll get a **Maven projects need to be imported** prompt.
Select **Import Changes** to download the dependencies from Maven Central for use in your project.

## Set up authentication

Register your application with Azure Active Directory so your app can view (but not update) your resources in Azure. This guide uses the [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2) , but you
can also [create the service principal via the web portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal).

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

Right-click the **src/main/resources** folder in the **Project** view. Select **New** > **File**, enter *auth.propeties* in the **Enter a new file name:** field, then select **OK**.

Enter the following text into this properties file:

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

Update the placeholder values in this property file with the following changes:

- subscription = use the *id* value from `az account list`
- client = use the *appId* value from the output taken from the service principal created in the previous step
- key = use the *password* value from the service principal creation output
- tenant - use the *tenant* value from the service principal creation output

## Create the application

Create a new class in IntelliJ that will hold the sample code below. Right-click the **src/main/java** folder in the **Project** view , then elect **New** > **Java Class**. 
Enter AzureMgmtDemo in the **Name:** field and select **OK**.

IntelliJ will open the source file for editing. Enter the following code into this file:

```java
import com.microsoft.azure.management.compute.VirtualMachine;
import com.microsoft.azure.management.Azure;
import com.microsoft.azure.PagedList;

import java.lang.ClassLoader;
import java.io.File;

public class AzureMgmtDemo {

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

Run the code by from the **Run** menu or selecting the green arrow to the left of the `main` method in the AzureMgmtDemo source. The output from the application will display in the **Run** window.

```
Found virtual machine myAzureVM with size Standard_DS1_v2 in resource group myazresgroup with state PowerState/running
```

## Learn more

See the How-Tos for more detailed samples on how to integrate Azure services into your application and manage your Azure resources from your code.

