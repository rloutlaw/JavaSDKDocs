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

This guide walks you through creating a command line tool with Eclipse that connects to Azure and lists virtual machines in a [resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview).

## Prerequisites

- [Eclipse](https://eclipse.org/downloads)
- [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2)

## Create your project

1. Create a simple Maven project in Eclipse. Select **File** > **New** > **Maven Project**. In **New Maven Project**, select **Create a simple project** and **Use default workspace location**. Select **Next**.
2. On the second page of **New Maven Project**, enter the following:

   - Group ID: `com.<username>.azure.mgmtdemo`  
   - Artifact ID: AzureMgmtDemo  
   - Version: 0.0.1-SNAPSHOT  
   - Packaging: jar  
   - Name: AzureMgmtDemo  

   ![Complete the configure project step in the New Maven project dialog](_img/create_maven_project.png)

3. Select **Finish**

## Import your dependencies

Double-click the **pom.xml** file in the Eclipse Project Explorer.   

![Select the pom.xml tab in Eclipse after double-clicking the pom.xml file in Project Explorer](_img/pom_xml_tab.png)

Select the **pom.xml** tab at the bottom o the window that appears and add the following XML after the `</name>` tag but before the `</project>` tag.

```XML
<dependencies>
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure</artifactId>
    <version>1.0.0-beta5</version>
</dependency>
<dependencies>
```

Eclipse will connect to Maven Central and download the Azure management libraries for Java along with their dependencies.

## Set up authentication

Register your application with Azure Active Directory so your app can read and modify resources in Azure. This example uses the [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2) , but you
can also [create the service principal via the web portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal).

### Create a service principal

Create a service principal to let your application authenticate with without directly using your account. Service principals let you authenticate apps without directly using 
 user accounts and identities. 

2. Log in using the Azure CLI 2.0 with `az login`. 
3. List the subscriptions for your account with `az account list`.
4. Select the subscription for your service principal to access with `az account set --subscription <subscription name>`. 
5. Create the service principal with `az ad sp create-for-rbac -n "AzureMgmtDemo" --role contributor --output json`. Copy the output from this command to a safe place as it has information you'll
need to use to authenticate your app with Azure.

### Authenticate with a credential file

In this example we will create a properties file with the service principal information and use that to authenticate our app.

Create an auth.properties in your src/main/resources folder in your Maven project in Eclipse. 
Right-click the **resources** folder and select **New File**, enter auth.propeties in the **File name:** field, then select OK.

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
			
			final String resourceGroup = "MY_RESOURCE_GROUP";
			
            // authenticate the app with Azure using the properties file created earlier
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

Update the value of the resourceGroup variable to a resource group in your subscription. Save your changes to the source code.

## Run the code 

Run the code by selecting the Run button in Eclipse (  , shortcut Ctrl+F11) or from the **Run** > **Run** menu. The output from the application will display in the **Console** window.

```
Found virtual machine myAzureVM with size Standard_DS1_v2 in resource group myazresgroup with state Succeeded
```

## Learn more

See the How-Tos for detailed samples on [managing virtual machines]() , [connecting to a Azure SQL database]() 