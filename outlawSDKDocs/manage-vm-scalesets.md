---
title: Manage virtual machine scale sets with Java | Microsoft Docs
description: Sample code to manage Azure scale sets in your Java code
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

## Create and manage Azure virtual machine scale sets in Java

[This sample](https://github.com/Azure-Samples/compute-java-manage-virtual-machine-scale-sets) code creates a new [virtual machine scale set](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview) using the Java management libraries. The sample covers both the creation of the scale set and the management of the scale set after it is created.

## Authenticate with Azure

[!INCLUDE [auth-include](_shared/auth-include.md)]

## Create a scale set

Before creating the scale set definition, first set up a virtual network for the scale set and a load balancer to distribute workload across scale set instances.

### Create a virtual network for the scale set

```java
   Network network = azure.networks().define(vnetName)
                    .withRegion(region)
                    .withNewResourceGroup(rgName)
                    .withAddressSpace("172.16.0.0/16")
                    .defineSubnet("Front-end")
                    .withAddressPrefix("172.16.1.0/24")
                    .attach()
                    .create();
```

### Create a load balancer to distribute load across the scale set

```java
LoadBalancer loadBalancer1 = azure.loadBalancers().define(loadBalancerName1)
                    .withRegion(region)
                    .withExistingResourceGroup(rgName)
                    .definePublicFrontend(frontendName)
                        .withExistingPublicIPAddress(publicIPAddress)
                        .attach()
                    // Add two backend one per rule
                    .defineBackend(backendPoolName1)
                        .attach()
                    .defineBackend(backendPoolName2)
                        .attach()
                    // Add two probes one per rule
                    .defineHttpProbe(httpProbe)
                        .withRequestPath("/")
                        .withPort(80)
                        .attach()
                    .defineHttpProbe(httpsProbe)
                        .withRequestPath("/")
                        .withPort(443)
                        .attach()

                    // Add two rules that uses above backend and probe
                    .defineLoadBalancingRule(httpLoadBalancingRule)
                        .withProtocol(TransportProtocol.TCP)
                        .withFrontend(frontendName)
                        .withFrontendPort(80)
                        .withProbe(httpProbe)
                        .withBackend(backendPoolName1)
                        .attach()
                    .defineLoadBalancingRule(httpsLoadBalancingRule)
                        .withProtocol(TransportProtocol.TCP)
                        .withFrontend(frontendName)
                        .withFrontendPort(443)
                        .withProbe(httpsProbe)
                        .withBackend(backendPoolName2)
                        .attach()

                    // Add nat pools to enable direct VM connectivity for
                    //  SSH to port 22 and TELNET to port 23
                    .defineInboundNatPool(natPool50XXto22)
                        .withProtocol(TransportProtocol.TCP)
                        .withFrontend(frontendName)
                        .withFrontendPortRange(5000, 5099)
                        .withBackendPort(22)
                        .attach()
                    .defineInboundNatPool(natPool60XXto23)
                        .withProtocol(TransportProtocol.TCP)
                        .withFrontend(frontendName)
                        .withFrontendPortRange(6000, 6099)
                        .withBackendPort(23)
                        .attach()
                    .create();
```

The example load balancer created here is lengthy but straightforward. The load balancer creates two backend network address pools-one to balance load across HTTP and the other to balance load across HTTPS. Requests to the "/" HTTP and HTTPS URI on the scale set instances are defined as health probes endpoints. NAT rules are set up on the load balancer for ports 22 and 23 on the scale set instances so they can be managed directly via telnet or SSH.

### Create the scale set
 
```java
 // Create a virtual machine scale set with three virtual machines
 // And, install Apache Web servers on them
VirtualMachineScaleSet virtualMachineScaleSet = azure.virtualMachineScaleSets().define(vmssName)
                    .withRegion(region)
                    .withExistingResourceGroup(rgName)
                    .withSku(VirtualMachineScaleSetSkuTypes.STANDARD_D3_V2)
                    .withExistingPrimaryNetworkSubnet(network, "Front-end")
                    .withExistingPrimaryInternetFacingLoadBalancer(loadBalancer1)
                    .withPrimaryInternetFacingLoadBalancerBackends(backendPoolName1, backendPoolName2)
                    .withPrimaryInternetFacingLoadBalancerInboundNatPools(natPool50XXto22, natPool60XXto23)
                    .withoutPrimaryInternalLoadBalancer()
                    .withPopularLinuxImage(KnownLinuxVirtualMachineImage.UBUNTU_SERVER_16_04_LTS)
                    .withRootUsername(userName)
                    .withSsh(sshKey)
                    .withNewDataDisk(100)
                    .withNewDataDisk(100, 1, CachingTypes.READ_WRITE)
                    .withNewDataDisk(100, 2, CachingTypes.READ_WRITE, StorageAccountTypes.STANDARD_LRS)
                    .withCapacity(3)
                    // Use a VM extension to install Apache Web servers
                    .defineNewExtension("CustomScriptForLinux")
                        .withPublisher("Microsoft.OSTCExtensions")
                        .withType("CustomScriptForLinux")
                        .withVersion("1.4")
                        .withMinorVersionAutoUpgrade()
                        .withPublicSetting("fileUris", fileUris)
                        .withPublicSetting("commandToExecute", installCommand)
                        .attach()
                    .create();
```

Use the virtual network definition and load balancer definitions created in the previous step to create a scale set with three Linux instances (`withCapacity(3)`) and three 100GB data disks each. The `defineNewExtension()` section installs the Apache web server on the instance members.

## Work with virtual machine scale set network interfaces

```java
            System.out.println("Listing scale set network interfaces ...");
            PagedList<VirtualMachineScaleSetNetworkInterface> vmssNics = virtualMachineScaleSet.listNetworkInterfaces();
            for (VirtualMachineScaleSetNetworkInterface vmssNic : vmssNics) {
                System.out.println(vmssNic.id());
            }
```

This sample simply prints out the ID of the network interface, but once you have the scale set network interface instance object you can use methods such as `ipConfigurations()` to query interface details.

## Get SSH connection strings for each scale set instance

```java
for (VirtualMachineScaleSetVM instance : virtualMachineScaleSet.virtualMachines().list()) {
                System.out.println("Scale set virtual machine instance #" + instance.instanceId());
                System.out.println(instance.id());
                PagedList<VirtualMachineScaleSetNetworkInterface> networkInterfaces = instance.listNetworkInterfaces();
                // Pick the first NIC on the instance and use its primary IP address
                VirtualMachineScaleSetNetworkInterface networkInterface = networkInterfaces.get(0);
                for (VirtualMachineScaleSetNicIPConfiguration ipConfig : networkInterface.ipConfigurations().values()) {
                    if (ipConfig.isPrimary()) {
                        List<LoadBalancerInboundNatRule> natRules = ipConfig.listAssociatedLoadBalancerInboundNatRules();
                        for (LoadBalancerInboundNatRule natRule : natRules) {
                            // search through the NAT rules for the rule matching the inbound SSH port on the backend for this IP address
                            if (natRule.backendPort() == 22) {
                                System.out.println("SSH connection string: " + userName + "@" + publicIPAddress.fqdn() + ":" + natRule.frontendPort());
                                break;
                            }
                        }
                        break;
                    }
                }
            }
```

## Stop the virtual machine scale set

```java
            // stop (not deallocate) all scale set instances
            virtualMachineScaleSet.powerOff();
```

Stopped instances have the virtual machine operating system shut down, but the scale set instances continue to reserve Azure compute, network, and storage resources used by the
instances when they were started.

## Deallocate the virtual machine scale set

```java
       // deallocate the virtual machine scale set
                 virtualMachineScaleSet.deallocate();
```

Deallocated scale set instances stop the operating system for the scale set instances as well as return the used compute and network resources (including any non-static IP addresses) used by the scale set instances. You continue to accrue charges for Azure storage used for the virtual machine's data and OS disks. 

## Start a virtual machine scale set

```java
    // start a dealloated or stopped virtual machine scale set
     virtualMachineScaleSet.start();
```

## Update the number of virtual machines instances in the scale set
```java
            // increase the number of virtual machine scale set instances to six from the
            // three in the create sample
            virtualMachineScaleSet.update()
                    .withCapacity(6)
                    .apply();
```


