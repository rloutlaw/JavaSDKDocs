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

# Manage Azure virtual networks with Java

[This sample] creates a [virtual network](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview) to connect your Azure compute resources on an isolated network segment you control.

## Sample code

[View the complete code sample on GitHub](https://github.com/Azure-Samples/network-java-manage-virtual-network/blob/master/src/main/java/com/microsoft/azure/management/network/samples/ManageVirtualNetwork.java).

### Authenticate with Azure

[!INCLUDE [auth-include](_shared/auth-include.md)]

### Create a network security group to block Internet traffic

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

This [network security group](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-nsg) blocks both inbound and outbound public Internet traffic. This network security group will not have an effect until it is applied to a subnet in your virtual network.

### Create a virtual network with two subnets

```java
// create the a virtual network with two subnets
// the backend one uses the existing network security group that blocks all internet traffic
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

The backend subnet is denied Internet access following the rules in the  network security group definition. The front end subnet uses the [default rules](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-nsg) which allow outbound traffic to the Internet.

### Create a network security group to allow inbound HTTP traffic
```java
// create a rule that allows inbound HTTP and blocks outbound Internet traffic
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

### Update a virtual network
```java
// update the front end subnet to use the rules in the new network security group
virtualNetwork1.update()
          .updateSubnet(vnet1FrontEndSubnetName)
          .withExistingNetworkSecurityGroup(frontEndSubnetNsg)
          .parent()
          .apply();
```

### Create a virtual machine on a subnet
```java
// use the existing virtual network and front-end subnet to attach the new VM to the network
VirtualMachine frontEndVM = azure.virtualMachines().define(frontEndVmName)
                    .withRegion(Region.US_EAST)
                    .withExistingResourceGroup(rgName)
                    .withExistingPrimaryNetwork(virtualNetwork1) 
                    .withSubnet(vnet1FrontEndSubnetName)
                    .withPrimaryPrivateIpAddressDynamic()
                    .withNewPrimaryPublicIpAddress(publicIpAddressLeafDnsForFrontEndVm)
                    .withPopularLinuxImage(KnownLinuxVirtualMachineImage.UBUNTU_SERVER_16_04_LTS)
                    .withRootUsername(userName)
                    .withSsh(sshKey)
                    .withSize(VirtualMachineSizeTypes.STANDARD_D3_V2)
                    .create();
```

`withExistingPrimaryNetwork()` and `withSubnet()` configure the virtual machine to use the network created previously.

### List virtual networks in a resource group
```java
// iterate over every virtual network in the resource group 
for (Network virtualNetwork : azure.networks().listByGroup(rgName)) {
    // for each subnet on the virtual network, log the network address prefix 
    for (Map.Entry<String, Subnet> entry : virtualNetwork.subnets().entrySet()) {
        String subnetName = entry.getKey();
        Subnet subnet = entry.getValue();
        System.out.println("Address prefix for subnet " + subnetName + " is " + subnet.addressPrefix());
    }
}
```       

### Delete a virtual network
```java
// if you already have the virtual network object it is easiest to delete by ID
azure.networks().deleteById(virtualNetwork1.id());

// Delete by resource group and name if you don't have the VirtualMachine object
azure.networks().deleteByGroup(rgName,vnetName1);
```

## Sample explanation

This sample creates a virtual network with two subnets and with one virtual machine each. The back subnet is completely cut off from the public Internet. The front-facing subnet accepts inbound HTTP traffic from the Internet. Every virtual machine on the subnet can still communicate with each other through the default network security group rules.

| Class used in sample | Notes |
|-------|-------|
| [com.microsoft.azure.management.network.Network](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.network._network) | Local object representation of the virtual network created from `azure.networks().define()...create()` . Update the Network object after it is created using the `update()...apply()` fluent chain.
| [com.microsoft.azure.management.network.Subnet](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.network._subnet) | Subnets are created on the virtual network when defining or updating the network using `withSubnet()`. Object representations of a Subnet can be retrieved from `Network.subnets().get()` or `Network.subnets().entrySet()` and queried for information about their properties.
| [com.microsoft.azure.management.network.NetworkSecurityGroup](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.network._network_security_group) | Created using the `azure.networkSecurityGroups().define()...create()` fluent chain and then applied to subnets through the updating or creating subnets in a virtual network. 

## Next steps

[!INCLUDE [next-steps](_shared/next-steps.md)]

Additional Azure storage samples can be found in the [Azure virtual netowrking documentation](https://docs.microsoft.com/en-us/azure/virtual-network/).