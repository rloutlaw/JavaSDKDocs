---
title: Manage Azure virtual networks with Java | Microsoft Docs
description: Sample code to manage Azure virtual networks in your Java code
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

# Manage Azure networks in Java

Create [virtual networks](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview) to partition and connect your Azure resources on the same logical network and create [network security groups]() to control how networks can access public or private networking resources.

This sample creates a single virtual network with two subnets. The back subnet is completely cut off from the public Internet. The front-facing subnet accepts inbound HTTP traffic from the Internet. Every virtual machine on the subnet can still communicate with each other through default network security group rules.

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

## Create a network security group isolating a subnet from Internet traffic

```java
 // this Network security group definition blocks out all traffic from and to the public Internet
            NetworkSecurityGroup backEndSubnetNsg = azure.networkSecurityGroups().define(vnet1BackEndSubnetNsgName)
                    .withRegion(Region.US_EAST)
                    .withNewResourceGroup(rgName)
                    .defineRule("DenyInternetInComing")
                        .denyInbound()
                        .fromAddress("INTERNET")
                        .fromAnyPort()
                        .toAnyAddress()
                        .toAnyPort()
                        .withAnyProtocol()
                        .attach()
                    .defineRule("DenyInternetOutGoing")
                        .denyOutbound()
                        .fromAnyAddress()
                        .fromAnyPort()
                        .toAddress("INTERNET")
                        .toAnyPort()
                        .withAnyProtocol()
                        .attach()
                    .create();
```

## Create a virtual network with two subnets

```java
// create the a virtual network with two subnets, with the backend one using the network security group
Network virtualNetwork1 = azure.networks().define(vnetName1)
                    .withRegion(Region.US_EAST)
                    .withExistingResourceGroup(rgName)
                    .withAddressSpace("192.168.0.0/16")
                    .withSubnet(vnet1FrontEndSubnetName, "192.168.1.0/24")
                    .defineSubnet(vnet1BackEndSubnetName)
                        .withAddressPrefix("192.168.2.0/24")
                        .withExistingNetworkSecurityGroup(backEndSubnetNsg)
                        .attach()
                    .create();
```

The backend subnet is denied Internet access through the network security group definition. The front end subnet uses the [default rules](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-nsg) which allow bidirectional traffic in the virtual network and outbound access to the Internet.

## Create a network security group to allow inbound HTTP traffic

```java
// create a rule that allows inbound HTTP from the Internet, and blocks all outbound Internet traffic
NetworkSecurityGroup frontEndSubnetNsg = azure.networkSecurityGroups().define(vnet1FrontEndSubnetNsgName)
                    .withRegion(Region.US_EAST)
                    .withExistingResourceGroup(rgName)
                    .defineRule("AllowHttpInComing")
                        .allowInbound()
                        .fromAddress("INTERNET")
                        .fromAnyPort()
                        .toAnyAddress()
                        .toPort(80)
                        .withProtocol(SecurityRuleProtocol.TCP)
                        .attach()
                    .defineRule("DenyInternetOutGoing")
                        .denyOutbound()
                        .fromAnyAddress()
                        .fromAnyPort()
                        .toAddress("INTERNET")
                        .toAnyPort()
                        .withAnyProtocol()
                        .attach()
                    .create();
```

This network security group does nothing until it is used to update or create a subnet.

## Update a virtual network

```java
// update the existing virtual network's front end subnet with the new seucirty rule to allow inbound HTTP traffic
virtualNetwork1.update()
                    .updateSubnet(vnet1FrontEndSubnetName)
                        .withExistingNetworkSecurityGroup(frontEndSubnetNsg)
                        .parent()
                    .apply();
```

## Create a virtual machine on a subnet

```java
            VirtualMachine frontEndVM = azure.virtualMachines().define(frontEndVmName)
                    .withRegion(Region.US_EAST)
                    .withExistingResourceGroup(rgName)
                    .withExistingPrimaryNetwork(virtualNetwork1) // use the existing virtual network and front-end subnet
                    .withSubnet(vnet1FrontEndSubnetName)
                    .withPrimaryPrivateIpAddressDynamic()
                    .withNewPrimaryPublicIpAddress(publicIpAddressLeafDnsForFrontEndVm)
                    .withPopularLinuxImage(KnownLinuxVirtualMachineImage.UBUNTU_SERVER_16_04_LTS)
                    .withRootUsername(userName)
                    .withSsh(sshKey)
                    .withSize(VirtualMachineSizeTypes.STANDARD_D3_V2)
                    .create();
```

## List virtual networks in a resource group

```java
            // the Utils.print() method is a utility method in the sample that writes information
            // about a resource to the console 
            for (Network virtualNetwork : azure.networks().listByGroup(rgName)) {
                Utils.print(virtualNetwork);
            }

            // iterate over every virtual network in the resource group 
            for (Network virtualNetwork : azure.networks().listByGroup(rgName)) {
                // for each subnet on the virtual network, log the address prefix 
                for (Map.Entry<String, Subnet> entry : virtualNetwork.subnets().entrySet()) {
                    String subnetName = entry.getKey();
                    Subnet subnet = entry.getValue();
                    System.out.println("Address prefix for subnet " + subnetName + " is " + subnet.addressPrefix());
                }
            }
```       

## Delete a virtual network

```java
// if you already have the virtual network object it is easiest to delete by group
azure.networks().deleteById(virtualNetwork1.id());

// same delete as above but by resource group and name
azure.networks().deleteByGroup(rgName,vnetName1);
```

