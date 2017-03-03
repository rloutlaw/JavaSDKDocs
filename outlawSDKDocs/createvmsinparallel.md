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

# Create virtual machines across mulitple regions in parallel with the Java SDK

This guide and associated sample creates mulitple virtual machines in different Azure regions in parallel using the [Azure management libraries for Java](https://github.com/Azure/azure-sdk-for-java). Each virtual machine is assigned its own public IP address and a storage account and virtual network in each region is created for the virtual machines to use. A traffic manager is also created to optimize performance across the virtual machines. 

Complete sample code for this scenario can be found on [Github](https://github.com/Azure-Samples/compute-java-create-virtual-machines-across-regions-in-parallel).

> [!WARNING]
> The sample creates a total of 48 VMs running Ubuntu 16.04 LTS of [size STANDARD_DS3_V2](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-sizes) across four regions. 
> Customize the sample to create a number of VMs of the size, region, and operating system environment you need.

## Authenticate with Azure

Create an [authentication file](https://github.com/Azure/azure-sdk-for-java/blob/master/AUTH.md) and export an environment variable `AZURE_AUTH_LOCATION` on the command line with the full path to the file.

```bash
export AZURE_AUTH_LOCATION=/Users/raisa/azure.auth
```

The authentication file is used to create the top level `Azure` object, which is used by the management libraries to define, create, and configure Azure resources.

[!code-java[VMsInParallel](sample-code/src/main/java/com/microsoft/azure/management/compute/samples/CreateVirtualMachinesInParallel.java#AUTH "Authentication")]

## Choose how many virtual machines to create

Create a simple `java.util.Map` object with each entry having a Region constant as the key and the number of virtual machines to run in the region as the value. If you don't define a Region as a key, no virtual machines will be created in that region.

```java
// a Map to store how many VMs to create in a region
Map<Region, Integer> virtualMachinesByLocation = new HashMap<Region, Integer>();

// each put() adds a region and number of VMs in that region to the Map
virtualMachinesByLocation.put(Region.US_EAST, 12);
virtualMachinesByLocation.put(Region.US_SOUTH_CENTRAL, 12);
virtualMachinesByLocation.put(Region.US_WEST, 12);
virtualMachinesByLocation.put(Region.US_NORTH_CENTRAL, 12);
```

This example creates 48 VMs total, twelve in each of the four regions listed.

## Create a resource group 

Create a new Azure resource group to logically assoicate the virtual machines and their resources.

```java
// create a resource group with a random valid name 
final String rgName = SdkContext.randomResourceName("rgCOPD", 24);
ResourceGroup resourceGroup = azure.resourceGroups().define(rgName)
                .withRegion(Region.US_EAST)
                .create();
```

The `SdkContext.randomResourceName()` method call generates a random valid name for the resource group with the specified prefix. Use any valid resource name instead if you'd prefer.

This guide and the [sample](https://github.com/Azure-Samples/compute-java-create-virtual-machines-across-regions-in-parallel) creates the following resources in this resource group:

- the virtual machines with public IP addresses
- a storage account and virtual network for each region for the virtual machines to use
- a [Traffic Manager](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview) to balance traffic across all virtual machines created

## Define the virtual machines and their resources

Create a `List<Creatable<VirtualMachine>>` object where each list item defines the properties of a single virtual machine.  When the list is fully populated, pass it to the API to create all items defined in the list in parallel.

```java
List<Creatable<VirtualMachine>> creatableVirtualMachines = new ArrayList<>();
    
    // outer loop iterates through the regions included in the map
    for (Map.Entry<Region, Integer> entry : virtualMachinesByLocation.entrySet()) {
        Region region = entry.getKey();
        Integer vmCount = entry.getValue();

        //=============================================================
        // Define one virtual network Creatable per region where all the virtual machines created are added to
        //
        String networkName = SdkContext.randomResourceName("vnetCOPD-", 20);
        Creatable<Network> networkCreatable = azure.networks().define(networkName)
                .withRegion(region)
                .withExistingResourceGroup(resourceGroup)
                .withAddressSpace("172.16.0.0/16");

        //=============================================================
        // Create one storage account Creatable per region for storing VMs disk
        //
        String storageAccountName = SdkContext.randomResourceName("stgcopd", 20);
        Creatable<StorageAccount> storageAccountCreatable = azure.storageAccounts().define(storageAccountName)
                .withRegion(region)
                .withExistingResourceGroup(resourceGroup);

        // generate a common prefix for every VM name
        String linuxVMNamePrefix = SdkContext.randomResourceName("vm-", 15);

            // inner loop iterates through every VM instance to create in the region
            for (int i = 1; i <= vmCount; i++) {

                //=============================================================
                // Create one public IP address Creatable for each VM
                //
                Creatable<PublicIpAddress> publicIpAddressCreatable = azure.publicIpAddresses()
                        .define(String.format("%s-%d", linuxVMNamePrefix, i))
                            .withRegion(region)
                            .withExistingResourceGroup(resourceGroup)
                            .withLeafDomainLabel(SdkContext.randomResourceName("pip", 10));

                publicIpCreatableKeys.add(publicIpAddressCreatable.key());

                //=============================================================
                // Create one virtual machine Creatable and add to the List 
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
                    creatableVirtualMachines.add(virtualMachineCreatable);
                }
            }
```

The outer `for` loop above iterates over each region, defining a virtual network and storage account for use by all virtual machines to be created in that region in this sample. 

The inner `for` loop defines a public IP address for the virtual machine and then defined the properties of the virtual machine itself, using the the definitions for the virtual network, storage account, and public IP address as parameters to the method chain that defines the virtual machine. The `sshKey` and `userName` variables are `String` constants containing your public SSH key for direct access to the virtual machines and the user name for the root account on the virtual machines.

The `createVirtualMachines.add(virtualMachineCreatable)` call adds the virtual machine definition to the List object used to create the VMs in parallel.

## Create the virtual machines

Create all of the virtual machines defined in the List in parallel:

```java
CreatedResources<VirtualMachine> virtualMachines = azure.virtualMachines()
                                                        .create(creatableVirtualMachines);
```

This returns a collection of the created virtual machines. Iterate over and inspect this object to verify the result of the batch creation of the virtual machine. 

```java
// list the IDs of each virtual machine created 
for (VirtualMachine virtualMachine : virtualMachines.values()) {
                System.out.println(virtualMachine.id());
}
```

## Create a Traffic Manager to distribute work

Create a [Traffic Manager](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview) to improve responsiveness and availability of the applications running on the virtual machines.
This traffic manager instance uses performance based routing and sets endpoints on the IP addresses of each of the virtual machines.

```java
String trafficManagerName = SdkContext.randomResourceName("tra", 15);
TrafficManagerProfile.DefinitionStages.WithEndpoint profileWithEndpoint = azure.trafficManagerProfiles()
     .define(trafficManagerName)
         .withExistingResourceGroup(resourceGroup)
         .withLeafDomainLabel(trafficManagerName)
         .withPerformanceBasedRouting();

int endpointPriority = 1;
TrafficManagerProfile.DefinitionStages.WithCreate profileWithCreate = null;
for (String publicIpResourceId : publicIpResourceIds) {
    String endpointName = String.format("azendpoint-%d", endpointPriority);
    if (endpointPriority == 1) {
    profileWithCreate = profileWithEndpoint.defineAzureTargetEndpoint(endpointName)
            .toResourceId(publicIpResourceId)
            .withRoutingPriority(endpointPriority)
            .attach();
    } else {
        profileWithCreate = profileWithCreate.defineAzureTargetEndpoint(endpointName)
            .toResourceId(publicIpResourceId)
            .withRoutingPriority(endpointPriority)
            .attach();
    }
      endpointPriority++;
}
TrafficManagerProfile trafficManagerProfile = profileWithCreate.create();
```

## Delete the resource group if something goes wrong
If there's an uncaught exception, print the stack trace and the sample will delete the resource group if it exists, deleting all Azure resources created by the sample.

```java
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

This is a useful exception catch block to start with in your own code since it ensures that if there is a unrecoverable failure, any resources created are cleaned up.