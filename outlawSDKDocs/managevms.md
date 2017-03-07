---
title: Manage Azure virtual machines from Java | Microsoft Docs
description: Sample code to manage Azure virtual machines from your Java applications
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

# Manage virtual machines in Java

This guide and the [associated sample code](https://github.com/Azure-Samples/compute-java-manage-vm/blob/master/src/main/java/com/microsoft/azure/management/compute/samples/ManageVirtualMachine.java) uses the [Azure management libraries for Java](https://github.com/Azure/azure-sdk-for-java) to perform the following common tasks:

- Create a virtual machine with an attached OS disk
- Start, stop, and restart a virtual machine
- List virtual machines in a resource group
- Update a virtual machine : resize a disk and attach or detach a disk
- Delete a virtual machine

## Authenticate with Azure

Create an [authentication file](https://github.com/Azure/azure-sdk-for-java/blob/master/AUTH.md) and export an environment variable `AZURE_AUTH_LOCATION` on the command line with the full path to the file.

```bash
export AZURE_AUTH_LOCATION=/Users/raisa/azure.auth
```

The authentication file is used to create the top level `Azure` object, which is used by the management libraries to define, create, and configure Azure resources.

```java
// pull in the location of the security file from the environment 
final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));

Azure azure = Azure
        .configure()
        .withLogLevel(LogLevel.NONE)
        .authenticate(credFile)
        .withDefaultSubscription();
```

## Create a Windows virtual machine

The process for creating a new virtual machine is straightfoward:
0. Use the `azure.virtualMachines().define()` method to define a virtual machine Creatable and call `with` methods to customize it.
0. Call the `create()` method to create the virtual machine.  

```java
            // Prepare a creatable data disk for VM
            //
            Creatable<Disk> dataDiskCreatable = azure.disks().define(Utils.createRandomName("dsk-"))
                    .withRegion(region)
                    .withExistingResourceGroup(rgName)
                    .withData()
                    .withSizeInGB(100);

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

This sample defines a `Disk` Creatable, then defines a virtual machine that will use the disk with `.withNewDataDisk(dataDiskCreatable)`. The `.create()` method creates the disk and attaches it to the virtual machine created at the same time. Creatable objects let you define resources and use them to assemble other resources without having to wait for their creation in Azure.

Learn more about using [Creatables](concepts.md#Creatbles) do define resources locally and create them only as needed when using the Java management libraries.

## Stop, start, and restart a virtual machine

Use `azure.virtualMachines.getById()` method to access its `VirtualMachine` object representation, then run the appropriate method on it to restart, stop (deallocate), or start it.

```java
// look up a virtual machine by ID and then restart, stop, and start it
azureVM = azure.getVirtualMachine.getById(windowsVmName);

azureVM.restart();
azureVM.powerOff();
azureVM.start();
```

## Add a virtual machine to an existing network

Extend your infrastructure by creating new virtual machines that use resources already in place.

```java
// Get the network the previously created windows virtual machine is using
Network network = windowsVM.getPrimaryNetworkInterface().primaryIPConfiguration().getNetwork();


            //=============================================================
            // Create a Linux VM in the same virtual network

            VirtualMachine linuxVM = azure.virtualMachines().define(linuxVmName)
                    .withRegion(region)
                    .withExistingResourceGroup(rgName)
                    .withExistingPrimaryNetwork(network)
                    .withSubnet("subnet1") // Referencing the default subnet name when no name specified at creation
                    .withPrimaryPrivateIPAddressDynamic()
                    .withoutPrimaryPublicIPAddress()
                    .withPopularLinuxImage(KnownLinuxVirtualMachineImage.UBUNTU_SERVER_16_04_LTS)
                    .withRootUsername(userName)
                    .withRootPassword(password)
                    .withSize(VirtualMachineSizeTypes.STANDARD_D3_V2)
                    .create();
```

## List virtual machines in the same resource group

```java
// get a list of VMs in the same resource group as windowsVM
String resourceGroupName = windowsVM.resourceGroupName();
PagedList<VirtualMachine> resourceGroupVMs = azure.virtualMachines().listByGroup(resourceGroupName); 

// for each vitual machine in the resource group
for (VirtualMachine virtualMachine : resourceGroupVMs) {
                Utils.print(virtualMachine);
            }
```

This is a read-only example of interacting with all virtual machines in a resource group.  Run update operations on each `VirtualMachine` inside the for loop if you need to make a change that spans the entire resource group.

## Update a virtual machine

Use the `update()` method and chain `with` methods after it to update a virtual machine. Finish the chain with `apply()`.

```java

                windowsVM.update()
                        .withNewDataDisk(10)
                        .apply();
```

This example adds a new 10GB data disk to the virtual machine created earlier in the sample.

> [!NOTE] 
> If the update requires the virtual machine to be stopped before it can be performed, the machine will be stopped then updated.


## Delete a virtual machine
```java
azure.virtualMachines().deleteById(windowsVM.id());
```

> Delete operations are synchronous in the code unless the corresponding async method is called and the Completable event is handled. If you don't want your code to block until 
> the delete completes, you'll need to use the async call instead and manage the returned Completeable object. Learn more about the Reactive async model used in the Java management libaries for Azure.

