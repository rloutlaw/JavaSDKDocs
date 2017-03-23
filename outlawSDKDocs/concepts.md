---
title: Azure management library concepts | Microsoft Docs
description: Understand patterns and concepts used by the Azure management libraries for Java
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

# Azure management library concepts

<a name="authenticate"></a>

## Authentication

The simplest way to authenticate is to create a properties file that contains credentials for an [Azure service principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-application-objects)

```text
# sample management library properties file
subscription=########-####-####-####-############
client=########-####-####-####-############
key=XXXXXXXXXXXXXXXX
tenant=########-####-####-####-############
managementURI=https\://management.core.windows.net/
baseURL=https\://management.azure.com/
authURL=https\://login.windows.net/
graphURL=https\://graph.windows.net/
```

- subscription: use the *id* value from `az account show` in the Azure CLI 2.0.
- client: use the *appId* value from the output taken from a service principal created to run the application. If you don't have a service principal for your app, [create one with the Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli).
- key: use the *password* value from the service principal output 
- tenant: use the *tenant* value from the service principal output

Save this file in a secure location on your system where your code can read it. Set an environment variable with the full path to the file in your shell:

```bash
AZURE_AUTH_LOCATION=/Users/raisa/azureauth.properties
```

Create the entry point `Azure` object to start working with the API:

```java
// pull in the location of the authenticaiton properties file from the environment 
final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));

Azure azure = Azure
        .configure()
        .withLogLevel(LogLevel.NONE)
        .authenticate(credFile)
        .withDefaultSubscription();
```

The samples use file-based authentication to keep the code compact and easy to follow. If you don't want to save the credentials in a file, you can load them from your Java code from a secure source (such as [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/) ) and initialize the `Azure` entry point using an [ApplicationTokenCredentials](https://github.com/Azure/azure-sdk-for-java/blob/master/AUTH.md#using-applicationtokencredentials) object.

<a name="Creatables"></a>

## Just in time resource creation with Creatables

One challenge when creating or updating an Azure resource is that those resources require other resources to exist before they can be created or updated. An example is reserving a public IP address and setting up a disk when creating a new virtual machine. You don't want to verify the reservation of the address or the creation of the disk-you just want to the virtual machine to have those resources.

Use `Creatable` objects to define Azure resources for use in your code but only create them when needed in Azure. Code written with `Creatable` objects offloads resource creation in the Azure environment to the management API, boosting performance. 

Generate `Creatable `objects through the resource collections' `define()` verb:

```java
Creatable<PublicIPAddress> publicIPAddressCreatable = azure.publicIPAddresses().define(publicIPAddressName)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup(rgName);
```

The Azure resource defined by the `Creatable` does not yet exist in your subscription. A `Creatable` is a local representation of a resource that the management API will create when its needed. Use this `Creatable` to define other Azure resources that need this resource. 

```java
Creatable<VirtualMachine> vmCreatable = azure.virtualMachines().define("creatableVM")
        .withNewPrimaryPublicIPAddress(publicIPAddressCreatable)
```

Create the resources in your Azure subscription using the  `create()` method for the resource collection. 

```java
CreatedResources<VirtualMachine> virtualMachine = azure.virtualMachines().create(vmCreatable);
```

Passing `Creatables` to `create()` calls returns a `CreatedResources` object instead of a single resource object.  The `CreatedResources` object lets you access all resources created by the `create()` call, not just the the type from the resource collection. To access the public IP address created in Azure for the virtual machine created in the above example:

```java
PublicIPAddress pip = (PublicIPAddress) virtualMachine.createdRelatedResource(publicIPAddressCreatable.key());
```

## Child resource collections

The management API has a single point of entry through the `com.microsoft.azure.management.Azure` object.  Select which type of resources to work with using the child resource collections in the `Azure` object, for example for SQL Database:

```java
SqlServer sqlServer = azure.sqlServers().define(sqlServerName)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup(rgName)
                    .withAdministratorLogin(administratorLogin)
                    .withAdministratorPassword(administratorPassword)
                    .create();
```

Most fluent conversations you have with the API starts with selecting the appropriate child resource collection for the Azure service or resources you need to work with.

## Lists and iterations

Every child resource collection has a `list()` method to return every instance of that resource in your current subscription. For example, `azure.sqlServers().list()` returns all SQL databases in the subscription.

Use the `listByGroup(String groupname)` method to scope the returned List to a specific [Azure resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#resource-groups).  

Iterate over the returned `PagedList` collection just as you would a normal `List`:

```java
PagedList<VirtualMachine> vms = azure.virtualMachines().list();
for (VirtualMachine vm : vms) {
    System.out.println("Found virtual machine with ID " + vm.id());
}
```

## Builders, not constructors

You do not call constructors to work with the management API. Use the fluent interface to build the API objects for your code. For example, the entry-point Azure object:

```java
Azure azure = Azure
                    .configure()
                    .withLogLevel(LogLevel.NONE)
                    .authenticate(credFile)
                    .withDefaultSubscription();
```


## Actionable verbs

When working with the management API, it's important to be aware of the method calls that you can make that will take action in Azure vs. methods calls that only manipulate the state of local objects. These methods use verbs as their names and are calleds from instances of child objects or from the main child object entry point itself (for example, `azure.virtualNetworks() ). Some of the

These methods work synchronously in your code. Async versions of these methods are provided that use the Reactive library and Observables. 

| Verb   |  Sample Usage |
|--------|---------------|
| create | `azure.virtualMachines().create(listOfVMCreatables)` |
| delete | `azure.disks().deleteById(id)` | 
| list   | `azure.sqlServers().list()` | 
| get    | `VirtualMachine vm  = azure.virtualMachines().getByResourceGroup(group, vmName)` |

Specific child resources have verbs that make sense in their contexts that work in the same way. For example:

```java
VirtualMachine vmToRestart = azure.getVirtualMachines().getById(id);
vmToStop.restart();
```
These child resource levels generally do not have asynchronous versions.

## Exception handling

The management API currently only throws a few typed exceptions, and most of these are subclasses of the `com.microsoft.rest.RestException` class. If you are looking to catch an exception specific to the management API, having a `catch (RestException exception)` block after the relevant try statement is recommended.

Open an [open an issue](https://github.com/Azure/azure-sdk-for-java/issues) on GitHub if you feel there's a scenario that really would benefit from a typed exception in the library.

## Returned object collections

The management library follows a convention for returned collections depending on the shape of the data returned:

- Lists: PagedLists are returned for unordered data for ease of iteration across each entry in the list.
- Maps: Maps are returned for objects that have unique keys, but not necessarily values. A good example of a Map would be a set of environment variables for a App Service app.
- Sets: Sets have unique keys and values. A good example of a Set would be networks attached to a virtual machine, which would have both a unique identifier and a unique network configuration.

Note the returned collection types when working to make assumptions about the returned data and keep your code from making logic checks about the returned data that might not be necessary.

## Logs and trace

Logging in the management API is supported at the REST-level calls that the Java libraries invoke. This logging uses the popular [SLF4J](https://www.slf4j.org/) library for logging and defaults to a simple scheme configured when you build the entry point `Azure` object using the `withLogLevel()` method. You can specify the following trace levels:

| Trace level | Logging enabled 
| ------------ | ---------------
| com.microsoft.rest.LogLevel.NONE | No output
| com.microsoft.rest.LogLevel.BASIC | Logs the URLs to underlying REST calls, response codes and times
| com.microsoft.rest.LogLevel.BODY | Everything in BASIC plus request and response bodies for the REST calls
| com.microsoft.rest.LogLevel.HEADERS | Everything in BASIC plus the request and response headers REST calls
| com.microsoft.rest.LogLevel.BODY_AND_HEADERS | Everything in both BODY and HEADERS log level

Bind a [SLF4J logging implementation](https://www.slf4j.org/manual.html) if you need to log output to a logging framework like [Log4J 2](https://logging.apache.org/log4j/2.x/)

