---
title: Manage Azure virtual machines with Java
description: Sample code to manage Azure virtual machines from your Java apps
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

# Manage virtual machines with Java

[This sample](https://github.com/Azure-Samples/compute-java-manage-vm/) uses the [Azure management libraries for Java](https://github.com/Azure/azure-sdk-for-java) to create and work with Azure virtual machines.

## Sample code

### Authenticate with Azure

[!INCLUDE [auth-include](_shared/auth-include.md)]

### Create a Windows virtual machine

```java
// Prepare a data disk for VM
Disk dataDisk = azure.disks().define(SdkContext.randomResourceName("dsk", 30))
                .withRegion(region)
                .withNewResourceGroup(rgName)
                .withData()
                .withSizeInGB(50)
                .create();

// create the windows virtual machine with the data disk            
VirtualMachine windowsVM = azure.virtualMachines().define(windowsVmName)
                .withRegion(region)
                .withNewResourceGroup(rgName)
                .withNewPrimaryNetwork("10.0.0.0/28")
                .withPrimaryPrivateIpAddressDynamic()
                .withoutPrimaryPublicIpAddress()
                .withPopularWindowsImage(KnownWindowsVirtualMachineImage.WINDOWS_SERVER_2012_R2_DATACENTER)
                .withAdminUsername(userName)
                .withAdminPassword(password)
                .withNewDataDisk(10)
                .withNewDataDisk(dataDiskCreatable)
                .withExistingDataDisk(dataDisk)
                .withSize(VirtualMachineSizeTypes.STANDARD_D3_V2)
                .create();
```

This code:
0. Defines a `Disk` Creatable with a 50GB size and random name to attach to the virtual machine.
0. Uses the `azure.virtualMachines().define()..create()` chain to create a Windows Server 2012 virtual machine. The sample creates the `Disk` defined in Azure at the same time as the virtual machine.

Learn more about using [Creatables](concepts.md#Creatbles) do define resources locally and create them only when needed.

### Stop, start, and restart a virtual machine

```java
// look up a virtual machine by its ID and then restart, stop, and start it
azureVM = azure.getVirtualMachine.getById(windowsVM.id());

azureVM.restart();
azureVM.powerOff(); // stops the operating system, but does not deallocate the VM resources
azureVM.start();
```

### Add a virtual machine to an existing network

```java
// Get the virtual network the current virtual machine is using
Network network = windowsVM.getPrimaryNetworkInterface().primaryIPConfiguration().getNetwork();

// Create a Linux VM in the same subnet
VirtualMachine linuxVM = azure.virtualMachines().define(linuxVmName)
               .withRegion(region)
               .withExistingResourceGroup(rgName)
               .withExistingPrimaryNetwork(network)
               .withSubnet("subnet1") // default subnet name when no name specified at creation
               .withPrimaryPrivateIPAddressDynamic()
               .withoutPrimaryPublicIPAddress()
               .withPopularLinuxImage(KnownLinuxVirtualMachineImage.UBUNTU_SERVER_16_04_LTS)
               .withRootUsername(userName)
               .withRootPassword(password)
               .withSize(VirtualMachineSizeTypes.STANDARD_D3_V2)
               .create();
```

### List virtual machines

```java
// get a list of VMs in the same resource group as an existing VM
String resourceGroupName = windowsVM.resourceGroupName();
PagedList<VirtualMachine> resourceGroupVMs = azure.virtualMachines().listByGroup(resourceGroupName); 

// for each vitual machine in the resource group, log their name and plan
for (VirtualMachine virtualMachine : azure.virtualMachines().listByGroup(resourceGroupName)) {
    System.out.println("VM " + virtualMachine.computerName() + " has plan " + virtualMachine.plan());
}
```

### Update a virtual machine

```java
// add a 10GB data disk to the virtual machine
windowsVM.update()
     .withNewDataDisk(10)
     .apply();
```

### Delete a virtual machine
```java
// delete by ID if you already are working with the VM object
azure.virtualMachines().deleteById(windowsVM.id());

// delete by resource group and name
azure.virtualMachines().deleteByGroup(rgName,windowsVmName);
```

## Sample explanation

[The sample code](https://github.com/Azure-Samples/compute-java-manage-vm/blob/master/src/main/java/com/microsoft/azure/management/compute/samples/ManageVirtualMachine.java) creates a Windows virtual machine with a 50GB data disk. The virtual machine is data disk configuration is then updated and the virtual machine restarted.

A Linux virtual machine is then created in the same virtual network. The code gets a list of virtual machines and then deletes both virtual machines before finishing.

| Class used in sample | Notes
|-------|-------|
| [com.microsoft.azure.management.compute.VirtualMachine](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.compute._virtual_machine) | Query properties and manage state of virtual machines. Retrieved in list form  with`azure.virtualMachines().list()` or by name or ID `azure.virtualMachines().getByGroup()`
| [com.microsoft.azure.management.compute.VirtualMachineSizeTypes](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.compute._virtual_machine_size_types) | Class with static values that map to [virtual machine size options](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/linux/), used by the `withSize()` method to define the resources allocated to the VM.
| [com.microsoft.azure.management.compute.Disk](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.compute._disk) | Create a disk to store data using `withData()` or operating system image using the appropriate `withLinux` or `withWindows` method when defining the disk. Attach disks to virtual machines either at the time of creation (`using withNewDataDisk` or `withExistingDataDisk`) or after creation by `update()..apply()` on the VirtualMachine object.
| [com.microsoft.azure.management.compute.DiskSkuTypes](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.compute._disk_sku_types) | Class with static values to define a disk with a standard or [premium](https://docs.microsoft.com/en-us/azure/storage/storage-premium-storage) storage plan.
| [com.microsoft.azure.management.compute.KnownLinuxVirtualMachineImage](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.compute._known_linux_virtual_machine_image) | Class with a set of Linux virtual machine options for use with the `withPopularLinuxImage()` method when defining a virtual machine.
| [com.microsoft.azure.management.compute.KnownWindowsVirtualMachineImage](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.compute._known_windows_virtual_machine_image) | Class with a set of Windows virtual machine image options for use with the `withPopularWindowsImage()` method when defining a virtual machine.

## Next steps

[!INCLUDE [next-steps](_shared/next-steps.md)]