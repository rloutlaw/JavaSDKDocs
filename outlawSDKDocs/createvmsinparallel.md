---
title: Create VMs across regions in parallel | Microsoft Docs
description: Sample code to create multiple VMs across different Azure regions in parallel
services: ''
documentationcenter: java
author: routlaw
manager: douge
editor: ''

ms.assetid: a90d2905-44eb-471e-a4a8-d52463cec72f
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: multiple
ms.devlang: Java
ms.topic: article
ms.date: 12/22/2016
ms.author: routlaw;asirveda

---

# Create virtual machines across multiple regions in parallel in Java

This guide and associated sample creates virtual machines in parallel across different Azure regions using the [Azure management libraries for Java](https://github.com/Azure/azure-sdk-for-java). A storage account and virtual network is created in each region and every virtual machine has its own unique public IP address. 

Complete sample code for this scenario can be found on [Github](https://github.com/Azure-Samples/compute-java-create-virtual-machines-across-regions-in-parallel).

> [!INFO]
> The sample creates a total of 48 VMs running Ubuntu 16.04 LTS of [size STANDARD_DS3_V2](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-sizes) across four regions. These virtual machines and their resources are deleted right before the sample code finishes.

## Authenticate with Azure

Create an [authentication file](https://github.com/Azure/azure-sdk-for-java/blob/master/AUTH.md) and set the environment variable `AZURE_AUTH_LOCATION` on the command line with the full path to the file.

```bash
export AZURE_AUTH_LOCATION=/Users/raisa/azure.auth
```

The authentication file is used to create the entry point `Azure` object used by the management libraries to define, create, and configure Azure resources.

```java
// pull in the location of the security file from the environment 
final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));

// generate the entry point Azure object
Azure azure = Azure
        .configure()
        .withLogLevel(LogLevel.NONE)
        .authenticate(credFile)
        .withDefaultSubscription();
```

## Choose how many virtual machines to create

```java
// create a java.util.Map to define how where and how many VMs to create 
Map<Region, Integer> virtualMachinesByLocation = new HashMap<Region, Integer>();

// each put() adds a region and number of VMs in that region into the Map
virtualMachinesByLocation.put(Region.US_EAST, 12);
virtualMachinesByLocation.put(Region.US_SOUTH_CENTRAL, 12);
virtualMachinesByLocation.put(Region.US_WEST, 12);
virtualMachinesByLocation.put(Region.US_NORTH_CENTRAL, 12);
```

This example creates 48 VMs total, twelve in each of the four regions added. If a region isn't defined with a key in the map, no VMs are created in that region.

## Create a resource group 

```java
// logically associate the resources in the sample into an Azure resource group
final String rgName = SdkContext.randomResourceName("rgCOPD", 24);
ResourceGroup resourceGroup = azure.resourceGroups().define(rgName)
                .withRegion(Region.US_EAST)
                .create();
```

The `SdkContext.randomResourceName()` method generates a random valid name for the resource group. Use any valid resource name instead if you'd prefer.

## Define the virtual machines and their resources

Create a `List<Creatable<VirtualMachine>>`  where each item in the list defines a single virtual machine. The virtual machines aren't created in Azure until `azure.virtualMachines().create()` is invoked with this list as a parameter.

Learn more about using [Creatables](concepts.md#Creatbles) do define resources locally and create them only when needed when using the Java management libraries.

```java
List<Creatable<VirtualMachine>> creatableVirtualMachines = new ArrayList<>();
    
    // outer loop: iterate through each region included in the map
    for (Map.Entry<Region, Integer> entry : virtualMachinesByLocation.entrySet()) {
        Region region = entry.getKey();
        Integer vmCount = entry.getValue();

        // Define one virtual network Creatable per region for the VMs to share
        String networkName = SdkContext.randomResourceName("vnetCOPD-", 20);
        Creatable<Network> networkCreatable = azure.networks().define(networkName)
                .withRegion(region)
                .withExistingResourceGroup(resourceGroup)
                .withAddressSpace("172.16.0.0/16");

        // Define one storage account Creatable per region for storing VM disks
        String storageAccountName = SdkContext.randomResourceName("stgcopd", 20);
        Creatable<StorageAccount> storageAccountCreatable = azure.storageAccounts().define(storageAccountName)
                .withRegion(region)
                .withExistingResourceGroup(resourceGroup);

        // generate a common prefix for every VM name
        String linuxVMNamePrefix = SdkContext.randomResourceName("vm-", 15);

            // inner loop: iterate once for every VM instance in the region
            for (int i = 1; i <= vmCount; i++) {

                // Create one public IP address Creatable for each VM
                Creatable<PublicIpAddress> publicIpAddressCreatable = azure.publicIpAddresses()
                        .define(String.format("%s-%d", linuxVMNamePrefix, i))
                            .withRegion(region)
                            .withExistingResourceGroup(resourceGroup)
                            .withLeafDomainLabel(SdkContext.randomResourceName("pip", 10));

                publicIpCreatableKeys.add(publicIpAddressCreatable.key());

                // Create one virtual machine Creatable 
                Creatable<VirtualMachine> virtualMachineCreatable = azure.virtualMachines()
                        .define(String.format("%s-%d", linuxVMNamePrefix, i))
                            .withRegion(region)
                            .withExistingResourceGroup(resourceGroup)
                            .withNewPrimaryNetwork(networkCreatable)
                            .withPrimaryPrivateIpAddressDynamic()
                            .withNewPrimaryPublicIpAddress(publicIpAddressCreatable)
                            .withPopularLinuxImage(KnownLinuxVirtualMachineImage.UBUNTU_SERVER_16_04_LTS)
                            .withRootUsername(userName)
                            .withSsh(sshKey)
                            .withSize(VirtualMachineSizeTypes.STANDARD_DS3_V2)
                            .withNewStorageAccount(storageAccountCreatable);
                    creatableVirtualMachines.add(virtualMachineCreatable); // add the virtual machine to the list
                }
            }
```

The outer `for` loop above iterates through each region, defining a virtual network and storage account for use by all virtual machines to be created in that region. 

The inner `for` loop defines a public IP address for the virtual machine and then defined the properties of the virtual machine itself, using the the definitions for the virtual network, storage account, and public IP address as parameters to the method chain that defines the virtual machine.

## Create the virtual machines

Create all of the virtual machines defined in the List in parallel:

```java
// create all virtual machines defined in the list, return all Creatable objects used
// including networks, public IP addresses, and storage accounts
CreatedResources<VirtualMachine> virtualMachines = azure.virtualMachines()
                                                        .create(creatableVirtualMachines);
```

Iterate over and inspect this object to verify the result of the batch creation of the virtual machine. 

```java
// list the IDs of each virtual machine created 
for (VirtualMachine virtualMachine : virtualMachines.values()) {
                System.out.println(virtualMachine.id());
}
```

The returned `CreatedResources` object that can be used to determine all resources created by the `create()` method, not just virtual machines. For example, the public IP addresses defined in the previous step whose Creatable keys were stored in `publicIpCreatableKey`:

```java
// call createdRelatedResource(key) to get the resources used to define the virtual machines
// the key is stored in a list when the Creatable was generated
for (String publicIpCreatableKey : publicIpCreatableKeys) {

                PublicIPAddress pip = (PublicIPAddress) virtualMachines.createdRelatedResource(publicIpCreatableKey);
            }
```

[Learn more](concepts.md#creatables) about working with CreatedResources in our [API concepts article](concepts.md).

## Delete the resource group 

```java
// finally block deletes the resource group even if the code before it throws an exception.
// deleting a resource group deletes all resources created in it
finally {
    try {
        System.out.println("Deleting Resource Group: " + rgName);
        azure.resourceGroups().deleteByName(rgName);
        System.out.println("Deleted Resource Group: " + rgName);
    } catch (NullPointerException npe) {
        System.out.println("Did not create any resources in Azure. No clean up is necessary");
    } catch (Exception g) {
        g.printStackTrace();
    }
}
```

This block ensures that any resources created in the sample are cleaned up before the code exits.