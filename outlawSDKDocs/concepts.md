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

## Authentication

The simplest way to authenticate is to create an external properties file that contains your application's credentials:

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

- subscription: use the *id* value from `az account show`
- client: use the *appId* value from the output taken from a service principal created to run the application. If you don't have a service principal yet, [use the Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli) to create one.
- key: use the *password* value from the service principal 
- tenant: use the *tenant* value from the service principal

Save this file in a secure location on your system where your Java code can read from it. Set an environment variable with its location in your shell:

```bash
AZURE_AUTH_LOCATION=/home/frank/secure/azureauth.properties
```

Use the properties file to create the entry point `Azure` object to start working with the library:

```java
// pull in the location of the authenticaiton properties file from the environment 
final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));

Azure azure = Azure
        .configure()
        .withLogLevel(LogLevel.NONE)
        .authenticate(credFile)
        .withDefaultSubscription();
```

This pattern is used in the samples since the code is compact and easy to follow. If you don't want to persist the credentials in a file, you can load them from your Java code from a secure source (such as [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/) ) and initialize the `Azure` entry point using an [ApplicationTokenCredentials](https://github.com/Azure/azure-sdk-for-java/blob/master/AUTH.md#using-applicationtokencredentials) object.


## Batch create resources with Creatables

One of the challenges when creating or updating a resource in Azure is that you might need other Azure resources to configure it. A good example is reserving a public IP address and setting up a disk for a new virtual machine. You don't want create and verify the creation of each intermediate resource-all you really care about is the final result (in our case, the virtual machine). 

Using asynchronous methods to create the resources is difficult because some Azures resources are fine to create in parallel but others need to be created in sequence, and it's not straightforward to know which is the case for your scenario. Concurrent code like this is also difficult to develop, debug, maintain, and extend later.

The management libraries provide a pattern to define Azure resources without immediately creating them using Creatable objects. Generate Creatable objects through the resource's `define()` verb, for example a public IP address:

```java
Creatable<PublicIPAddress> publicIPAddressCreatable = azure.publicIPAddresses().define(publicIPAddressName)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup(rgName);
```

At this point, the Azure resource defined by the Creatable does not yet exist in your Azure subscription-it is only a representation of a resource that the management API can create later. Use this Creatable to define other Azure objects that use this resource. These objects can also be Creatables:

```java
Creatable<VirtualMachine> vmCreatable = azure.virtualMachines().define("creatableVM")
        .withNewPrimaryPublicIPAddress(publicIPAddressCreatable)
```

In this manner you can set up a batch of Azure resources defined in your code for your configuration and use them to configure your environments without managing the creation of the resources every time. When you're ready in your code to create the resources in Azure, generate the Creatables using the `create()` method for the top-level resource type you are working with. To continue the virtual machine example:

```java
CreatedResources<VirtualMachine> virtualMachine = azure.virtualMachines().create(vmCreatable);
```

`create()` calls that take a Creatable return a `CreatedResources` generic instead of just a type of the created Azure resource. This is important since when a Creatable is created, any Creatables (such as the IP address in the example) used in defining the resource are also created in Azure. The `CreatedResources` object lets you access all resources created by the `create()` call. For example, to access the actual IP address created for the virtual machine:

```java
PublicIPAddress pip = (PublicIPAddress) virtualMachine.createdRelatedResource(publicIPAddressCreatable.key());
```

Use Creatables in your code to define resources locally and batch their creation in parallel in Azure. Creatables keeps your code synchronous and easy to follow and the API takes care of maximizing parallel resource creation, reducing the amount of calls to the Azure backend helping your code to complete faster.

## Child resources

The management libraries have a single point of entry in the `Azure` object you create an instance of. From the `Azure` object, you then select which type of resources to work with , for example:

azure.virtualMachines() 
azure.storageAccounts()
azure.sqlServers()

These child resources are what you define locally and interact with using the API's [verbs](#verbs). Every fluent conversation you have with the API starts with selecting the appropriate child resource for the Azure service or resources you need to work with.

## Lists and iterations

Two lists methods are available for every child resource and return a uniform `List` collection to iterate through.

`list()` against a child resource returns a `PagedList` collection for all resources of that type in the Azure subscription used to create the entry-point `Azure` object. For example:

`azure.sqlServers().list()` returns all SQL databases in the subscription.

Scope the returned resources to a specific Azure resource group by using the `listByGroup(String groupname)` method. This call only returns the resources in a specific resource group in the subscription. 

Returned `PagedList` objects can be iterated over just like a normal `List` collection:

```java
PagedList<VirtualMachine> vms = azure.virtualMachines().list();
for (VirtualMachine vm : vms) {
    System.out.println("Found virtual machine with ID " + vm.id());
}
```

## Builders, not constructors

You don't need to call constructors to work with the management API as it uses a Builder pattern with a fluent interface to create objects for you to use. For example, the entry-point Azure object:

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

The management API currently only throws a few typed exceptions, and most of these are subclasses of the `com.microsoft.rest.RestException` class. If you are looking to catch an exception specific to the management API, having a `catch (RestException exception` block after the relevant try statement is recommended.

Open an [open an issue](https://github.com/Azure/azure-sdk-for-java/issues) on GitHub if you feel there's a scenario that really would benefit from a typed exception in the library.

## Returned object collections

The management library follows a convention for returned collections depending on the shape of the data returned:

- Lists: PagedLists are returned for unordered data for ease of iteration across each entry in the list.
- Maps: Maps are returned for objects that have unique keys, but not necessarily values. A good example of a Map would be a set of environment variables for a App Service app.
- Sets: Sets have unique keys and values. A good example of a Set would be networks attached to a virtual machine, which would have both a unique identifier and a unique network configuration.

Note the returned collection types when working to make assumptions about the returned data and keep your code from making logic checks about the returned data that might not be necessary.

## Logs and trace

Logging in the management API is supported at the REST-level calls that the Java libraries invoke. This logging uses the popular [SLF4J](https://www.slf4j.org/) library for logging and defaults to a simple scheme configured when you build the entry point `Azure` object using the `withLogLevel()` method. You can specify the following trace levels:

| Trace level | Logging enabled | 
| com.microsoft.rest.LogLevel.NONE | No output |
| com.microsoft.rest.LogLevel.BASIC | Logs the URLs to underlying REST calls, response codes and times |
| com.microsoft.rest.LogLevel.BODY | Everything in BASIC plus request and response bodies for the REST calls |
| com.microsoft.rest.LogLevel.HEADERS | Everything in BASIC plus the request and response headers REST calls | 
| com.microsoft.rest.LogLevel.BODY_AND_HEADERS | Everything in both BODY and HEADERS log level | 

Bind a [SLF4J logging implementations](https://www.slf4j.org/manual.html) if you need to log output to a logging framework like [Log4J 2](https://logging.apache.org/log4j/2.x/)

