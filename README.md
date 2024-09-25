# Traffic-Routing-in-Microsoft-Azure : Subnet to Network Virtual Appliance (NVA)

This lab demonstrates how User-Defined Routes can be created to route traffic between subnets through a network virtual appliance (NVA). In this scenario, any outbound traffic flow from the Public subnet will be redirected to the NVA before being allowed to communicate with the Private subnet. The following steps were executed to accomplish this task:

1. Created a virtual network
2. Created and configured an NVA virtual machine
3. Created a route table
4. Created a route
5. Associated a route table to a subnet
6. Created public and private virtual machines
7. Tested the routing of network traffic


![image](https://miro.medium.com/v2/resize:fit:640/format:webp/1*B0nPP-6nYVOssg5Q6G4u-w.png)

1. Create Virtual Network and Subnets
In Step 1, I created a virtual network with an address space of 10.0.0.0/16 and three subnets:

Public-Subnet (10.0.0.0/24)
Private-Subnet (10.0.1.0/24)
DMZ-Subnet (10.0.2.0/24)

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*aFpm1blHDXqkDar5bXTn1g.png)
![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*aFpm1blHDXqkDar5bXTn1g.png)

A dedicated subnet, AzureBastionSubnet, was created when I deployed an Azure BastionHost. The Azure Bastion is a managed service that provides us with secure access to our Azure networks and devices without needing to create and use a public IP address. I will utilize the Bastion to connect to the Azure environment for testing this demonstration.

!|Image](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*tW31PkWdvRcCvTzGXT2G9g.png)

2. Created a Network Virtual Appliance (NVA) that routes traffic
A virtual appliance is a virtual machine that typically runs a network application, such as a firewall. To learn about various pre-configured network virtual appliances you can deploy in a virtual network, see the Azure Marketplace.

When you create a route with the virtual appliance hop type, you also specify a next hop IP address. In this step, I deployed a virtual machine into the DMZ subnet of the virtual network. This VM will serve as the Network Virtual Appliance to help with the network traffic routing function that I want to achieve.

![Image](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*P60vg68B3PTqAyNaqYU4Vw.png)

The NVA has a private IP address of 10.0.2.4

![Image](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*_qI6T4d6Nd5V5Oy5JeDsAg.png)

