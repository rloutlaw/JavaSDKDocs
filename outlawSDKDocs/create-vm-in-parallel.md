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

# Create virtual machines across multiple regions with Java

[This sample](https://github.com/Azure-Samples/compute-java-create-virtual-machines-across-regions-in-parallel) creates virtual machines in parallel across different Azure regions using the [Azure management libraries for Java](https://github.com/Azure/azure-sdk-for-java).

>[!INFO]
> The sample creates a total of 48 VMs running Ubuntu 16.04 LTS of [size STANDARD_DS3_V2](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-sizes) across four regions. The sample code deletes these virtual machines before exiting.

## Sample code

View the [complete code sample on GitHub](https://github.com/Azure-Samples/compute-java-create-virtual-machines-across-regions-in-parallel/blob/master/src/main/java/com/microsoft/azure/management/compute/samples/CreateVirtualMachinesInParallel.java).

### Authenticate with Azure

[!INCLUDE [auth-include](_shared/auth-include.md)]

### Set locations and counts for the virtual machines

```java
// use a Map to define how where and how many VMs to create 
Map<Region, Integer> virtualMachinesByLocation = new HashMap<Region, Integer>();

// create 12 virtual machines in four regions
virtualMachinesByLocation.put(Region.US_EAST, 12);
virtualMachinesByLocation.put(Region.US_SOUTH_CENTRAL, 12);
virtualMachinesByLocation.put(Region.US_WEST, 12);
virtualMachinesByLocation.put(Region.US_NORTH_CENTRAL, 12);
```

### Create a resource group 

```java
// logically associate the resources in the sample into a randomly named resource group
final String rgName = SdkContext.randomResourceName("rgCOPD", 24);
ResourceGroup resourceGroup = azure.resourceGroups().define(rgName)
                .withRegion(Region.US_EAST)
                .create();
```

### Define the virtual machines
```java
// list to store the VirtualMachine definitions
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
        // add the virtual machine Creatable to the list     
        creatableVirtualMachines.add(virtualMachineCreatable); 
     }
}
```

The outer `for` loop above iterates through each region, defining a virtual network Creatable and storage account Creatable for use by all virtual machines in that region. Learn more about using [Creatables](concepts.md#Creatbles) to create resources only as needed when using the management libraries.

The inner `for` loop gets a public IP address Creatable for the virtual machine and then defines a virtual machine Creatable using the Creatables for the virtual network, storage account, and public IP address defined previously.  This VirtualMachine Creatable is then added to the `creatableVirtualMachines` list.

### Create the virtual machines

```java
// create all virtual machines defined in the list, return all Creatable objects used
// including networks, public IP addresses, and storage accounts
CreatedResources<VirtualMachine> virtualMachines = azure.virtualMachines()

// list the IDs of each virtual machine created 
for (VirtualMachine virtualMachine : virtualMachines.values()) {
    System.out.println(virtualMachine.id());
}

// call createdRelatedResource(key) to get the resources used to define the virtual machines. You need to save
// key String at the time you define the Creatble to use CreatedResources
for (String publicIpCreatableKey : publicIpCreatableKeys) {
    PublicIPAddress pip = (PublicIPAddress) virtualMachines.createdRelatedResource(publicIpCreatableKey);
}
```

The virtual machines defined in the list the aren't created in Azure until the code calls `azure.virtualMachines().create()` passing the `creatableVirtualMachines` List as a parameter.

Use the returned `CreatedResources` object to access any resources created in the Azure subscription during the the `create()` method, not just the returned `VirtualMachine` type. Cast the returned value from `createdRelatedResources()` to the correct type. 

Learn more about working with `Creatables` and `CreatedResources` in our [API concepts article](concepts.md).

### Delete the resource group 

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

This block deletes resources created in the sample before the sample exits.

## Sample explanation

View the complete sample code on [Github](https://github.com/Azure-Samples/compute-java-create-virtual-machines-across-regions-in-parallel).

The sample uses `Creatable` objects to define a virtual network and storage account for each region hosting the virtual machines. `Creatable` objects are then defined for the public IP address for each virtual machine. The sample defines the virtual machines using these `Creatable` objects, and sample adds the VM definition to the `virtualMachineCreatable` list.

After the code adds every virtual machine definition to the list, `azure.virtualMachines().create(creatableVirtualMachines)` creates each virtual machine in parallel in Azure.

The sample code then gets the IP addresses for all of the created virtual machines from the returned CreatedResources object to create a [Traffic Manager](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview) to distribute load across the virtual machines. 

The `finally` block deletes the resources from your Azure subscription even in the case of an error.

| Class used in sample | Notes
|-------|-------|
| [com.microsoft.azure.management.compute.VirtualMachine](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.compute._virtual_machine) | Query properties and manage state of virtual machines. Retrieved in list form  with`azure.virtualMachines().list()` or by name or ID `azure.virtualMachines().getByGroup()`
| [com.microsoft.azure.management.compute.VirtualMachineSizeTypes](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.compute._virtual_machine_size_types) | Class with static values that map to [virtual machine size options]
| [com.microsoft.azure.management.network.PublicIpAddress](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.network._public_i_p_address) | Defined, but not immediately created, for each virtual machine through `azure.publicIpAddresses().define()`. Store the key for each `Creatable` and retrieve later through `createdRelatedResource()`
| [com.microsoft.azure.management.compute.KnownLinuxVirtualMachineImage](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.compute._known_linux_virtual_machine_image) | Class with a set of Linux virtual machine options for use with the `withPopularLinuxImage()` method when defining a virtual machine.
| [com.microsoft.azure.management.network.Network](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.network._network) | One virtual network Creatable defined for each region through  `azure.networks().define()` . 

## Next steps

[!INCLUDE [next-steps](_shared/next-steps.md)]