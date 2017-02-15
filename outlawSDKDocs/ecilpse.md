---
title: Get started with Azure development in Eclipse | Microsoft Docs
description: Learn how to start working with the Azure and Java in the Eclipse IDE.
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
# Get started with Azure development in Java with Eclipse

This guide will walk you through creating a command line tool with Eclipse that connects to Azure and lists virtual machines in a given resource group.

## Prerequisites

- Eclipse running with a Java 8 JDK.
- [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2)

## Create your project

1. Create a simple Maven project. Select **File** > **New** > **Maven Project**. In **New Maven Project**, select **Create a simple project** and **Use default workspace location.**
2. On the second page of **New Maven Project**, enter the following:
- Group ID: `com.<username>.azure.mgmtdemo`  
- Artifact ID: AzureMgmtDemo  
- Version: 0.0.1-SNAPSHOT  
- Packaging: jar  
- Name: AzureMgmtDemo  
Select **Finish**


## Import your dependencies

Double-click the **pom.xml** file in your project. Select the **pom.xml** tab and add the following before the `</project>` tag at the end of the file:

```XML
<dependencies>
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure</artifactId>
    <version>1.0.0-beta5</version>
</dependency>
<dependencies>
```

Eclipse will connect to Maven Central and download the Azure management libraries for Java and their dependencies.

## Set up authentication

Register your application with Azure Active Directory so your app can read and modify resources in Azure.

### Create a service principal

Create a service principal to let your application authenticate with without using your account. Service principals let you manage application access separately from your 
actual user accounts and identities. 

2. Log in using the Azure CLI 2.0 with `az login`. 
3. List the subscriptions for your account with `az account list`.
4. Select the subscription for your service principal to access with `az account set --subscription <subscription name>`. 
5. Create the service principal with `az ad sp create-for-rbac -n "AzureMgmtDemo" --role contributor --output json`. Copy the output from this command to a safe place as it has information you'll
need to use to authenticate your app with Azure.

### Authenticate with a credential file

In this example we will create a properties file with the service principal information and use that to authenticate our app.

Create a file auth.properties in your src/main/resources folder in your Maven project in Eclipse. Right-click the **resources** folder and select **New File**, enter auth.propeties in the **File name:** field, then select OK.

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

The subscription, client, key, and tenant information is taken from the output from when you created the service principal.

## Connect to Azure 

Create a new class in the src/main/java path in Eclipse (Right click the path, select **New** > **Class**). Enter AzureMgmtDemo in the **Name:** field and select the option to create
a main method under **Which method stubs would you like to create?**. Leave the rest of the options as is and select **Finish**. 

Eclipse will open the class file for editing. Enter the following code into the file:

```java
import com.microsoft.azure.management.compute.VirtualMachine;
import com.microsoft.azure.management.Azure;
import com.microsoft.azure.PagedList;
import java.lang.ClassLoader;
import java.io.File;

public class AzureMgmtDemo {

	public static void main(String[] args) {
		
		try {
			
			String resourceGroup = "MY_RESOURCE_GROUP";
			
            // authenticate the app with Azure using the properties file generated in the previous step 
            ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
            File authfile = new File(classLoader.getResource("auth.properties").getFile());
			Azure azure = Azure.authenticate(authfile).withDefaultSubscription();
			
            // get a list of VMs running in the Azure resource group passed on the command line
			PagedList<VirtualMachine> vmlist = azure.virtualMachines().listByGroup(resourceGroup);
			
			for(VirtualMachine vm : vmlist) {
				if (vm != null) {
					// write VM information to system out
					System.out.println("Found virtual machine " + vm.name() 
				        + " with size " + vm.size() + " in resource group " + resourceGroup 
						+ " with state " + vm.provisioningState());
				}
			}
		}			
		catch (Exception e){
			System.err.println("Error listing virtual machines in resource group " 
			    + resourceGroup + " : " + e.getMessage());
		}
	}
}
```

This code reads in the properties file created in the previous step. Then it gets a list of VMs for a resource group, then writes out some information about those VMs to the console.
Management tasks in Java flow down from the top-level Azure object-you identify which resources you want to work with, then call the methods to read, update, or delete resources of that type. 
Refer to our How-Tos for walkthroughs and sample code to [create a virtual machine](), [manage Azure storage](), or [work with your Azure SQL databases]().

