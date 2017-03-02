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

# Create virtual machines across mulitple regions in parallel 

This sample creates mulitple virtual machines in different Azure regions in parallel. Each virtual machine is assigned its own public IP address and a storage account in each region is created for the VMs to use. A traffic manager profile is also created for the virtual machines. All resources created belong to a single Azure resource group.

Sample code for this scenario can be found on [Github](https://github.com/Azure-Samples/compute-java-create-virtual-machines-across-regions-in-parallel).

> [!WARNING]
> The sample code creates a total of 48 VMs running Ubuntu 16.04 LTS of [size STANDARD_DS3_V2](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-sizes) across four regions. 
> Customize the sample to create a number of VMs of the size and operating system environment that you need.

## Authenticate with Azure

Create an [authentication file](https://github.com/Azure/azure-sdk-for-java/blob/master/AUTH.md) and export an environment variable `AZURE_AUTH_LOCATION` on the command line with the full path to the file.

```bash
export AZURE_AUTH_LOCATION=/Users/raisa/azure.auth
```

The authentication file is used to create the top level `Azure` object, which is used by the Java SDK to generate and configure resources.

```java
            final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));

            Azure azure = Azure
                    .configure()
                    .withLogLevel(LogLevel.NONE)
                    .authenticate(credFile)
                    .withDefaultSubscription();
```

## Choose where and how many virtual machines to create

Create a simple Map object with each entry being an Azure region as the key and the number of virtual machines in that region as the value. 

```java
Map<Region, Integer> virtualMachinesByLocation = new HashMap<Region, Integer>();
// each put() adds a region and count to the Map
virtualMachinesByLocation.put(Region.US_EAST, 12);
virtualMachinesByLocation.put(Region.US_SOUTH_CENTRAL, 12);
virtualMachinesByLocation.put(Region.US_WEST, 12);
virtualMachinesByLocation.put(Region.US_NORTH_CENTRAL, 12);
```

## Create the resource group 

Create a new Azure resource group to group together the virtual machines and their dependent resources.


```java
        final String rgName = SdkContext.randomResourceName("rgCOPD", 24);
        ResourceGroup resourceGroup = azure.resourceGroups().define(rgName)
                    .withRegion(Region.US_EAST)
                    .create();
```

This guide and the complete sample creates the following resources in this resource group:

- the virtual machines with public IP addresses
- a storage account and virtual network for each region for the virtual machines to use
- a traffic manager to balance traffic across all VMs created

The `SdkContext.randomResourceName()` method call generates a random valid name for the resource group with the specified prefix. Use any valid resource name instead if you'd prefer.

## Generate a list of Creatables for the virtual machines and their resources

Create a List of `Creatable` objects where each list item defines the properties of a single virtual machine. Once the list of Creatable objects is populated, you instruct the API to create all items in the List in parallel.

```java
List<Creatable<VirtualMachine>> creatableVirtualMachines = new ArrayList<>();

            for (Map.Entry<Region, Integer> entry : virtualMachinesByLocation.entrySet()) {
                Region region = entry.getKey();
                Integer vmCount = entry.getValue();

                //=============================================================
                // Create one network creatable per region
                // Prepare Creatable Network definition (Where all the virtual machines in this region get added to)
                //
                String networkName = SdkContext.randomResourceName("vnetCOPD-", 20);
                Creatable<Network> networkCreatable = azure.networks().define(networkName)
                        .withRegion(region)
                        .withExistingResourceGroup(resourceGroup)
                        .withAddressSpace("172.16.0.0/16");

                //=============================================================
                // Create 1 storage creatable per region (For storing VMs disk)
                //
                String storageAccountName = SdkContext.randomResourceName("stgcopd", 20);
                Creatable<StorageAccount> storageAccountCreatable = azure.storageAccounts().define(storageAccountName)
                        .withRegion(region)
                        .withExistingResourceGroup(resourceGroup);

                String linuxVMNamePrefix = SdkContext.randomResourceName("vm-", 15);
                for (int i = 1; i <= vmCount; i++) {

                    //=============================================================
                    // Create 1 public IP address creatable for each VM
                    //
                    Creatable<PublicIpAddress> publicIpAddressCreatable = azure.publicIpAddresses()
                            .define(String.format("%s-%d", linuxVMNamePrefix, i))
                                .withRegion(region)
                                .withExistingResourceGroup(resourceGroup)
                                .withLeafDomainLabel(SdkContext.randomResourceName("pip", 10));

                    publicIpCreatableKeys.add(publicIpAddressCreatable.key());

                    //=============================================================
                    // Create 1 virtual machine creatable, repeat loop vmCount times
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

The outer loop above iterates through each region, defining a Creatable for a virtual network and storage account for use by all VMs to be created in that region. The inner loop defines a Creatable for the public IP address for the virtual machine and then a Creatable for the virtual machine itself, using the Creatables for virtual network, storage account, and public IP address as parameters to the method chain to configure the virtual machine Creatable. Note that the `sshKey` and `userName` variables used here are Strings to the public SSH key configured for direct access to the VMs and the user name for the root account on the virtual machine.

The `createVirtualMachines.add(virtualMachineCreatable)` call adds the virtual machine Creatable to the List object used later to create the VMs in parallel.


### Create the VMs

With the list of Creatables populated, this line of code creates all of the VMs defined in the list in parallel:

```java
CreatedResources<VirtualMachine> virtualMachines = azure.virtualMachines().create(creatableVirtualMachines);
```

This returns the results of the batch creation of the virtual machines. Iterate over this object to inspect the result of the batch creation of the VMs. For example:

```java
for (VirtualMachine virtualMachine : virtualMachines.values()) {
                System.out.println(virtualMachine.id());
            }
```

This lists the IDs of each virtual machine created to the console.

## Create a traffic manager across all the IP addresses of all the VMs created

Create a Traffic Manager to improve responsiveness and availability of the applications running on the VMs.
This traffic manager instance uses performance based routing and sets endpoints on the IP addresses of the VMs created.

```java
String trafficManagerName = SdkContext.randomResourceName("tra", 15);
            TrafficManagerProfile.DefinitionStages.WithEndpoint profileWithEndpoint = azure.trafficManagerProfiles().define(trafficManagerName)
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