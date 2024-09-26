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

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*_qI6T4d6Nd5V5Oy5JeDsAg.png)

## Turn on IP Forwarding
To route traffic through the NVA, turn on IP forwarding in Azure and in the operating system of myVMNVA virtual machine. Once IP forwarding is enabled, any traffic received by myVMNVA VM that’s destined for a different IP address, won’t be dropped and will be forwarded to the correct destination.

Turn on IP forwarding in Azur

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*85Cw_4Re2KEufwFQqqsIlg.png)

## Turn on IP forwarding in the operating system of the NVA

Use the BastionHost to connect to myVMNVA, and then execute the PowerShell command listed below to enable IP forwarding in the Windows Operating System

> Set-ItemProperty -PathHKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters -NameIpEnableRouter -Value1

![Imqge](https://miro.medium.com/v2/resize:fit:720/format:webp/1*QGxLZfRJHVDJwkc38BvK3Q.png)

Then, Restart the myVMNVA virtual machine by executing the PowerShell command listed below.

> Restart-Computer

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*tWzsSkKfOs2l3zPEk1Tm5g.png)

3. Create a route table
Routes are paths through which traffic can flow and these routes are referenced in Azure when determining the next hop for routing traffic through the network. In this step, I created a route table, named myRouteTablePublic, that will be used to hold the routes that I configure for the subnets of the virtual network in the next step.

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*xvZF7Kf3BeUgtnV-kenk1Q.png)

4. Create a route
Next, I created a route and added it to the route table. Here, I created a route named “ToPrivateSubnet”. This route gives the instructions that any traffic that is destined to the private subnet (10.0.1.0/24) will be redirected such that it will instead have a Next hop type of “Virtual appliance”, which has a Next hop IP address of 10.0.2.4. This is the IP address that is assigned to the NVA.

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*wwVK8HQ54uVbhY3WBn1A3w.png)

5. Associate a route table to a subnet
Next, we associate the new route table with a subnet so that the subnet will have the new instructions for routing outbound traffic that is destined for the private subnet. The route that has been defined in this route table will override the default System Route that would have otherwise allowed the Public-Subnet to communicate with the Private-Subnet automatically
![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Nxna-HMMw6ExfSIEHMZAmg.png)

6. Deploy virtual machines (VMs) into different subnets
Next, I deployed virtual machines into the two other subnets, Private-Subnet and Public-Subnet.

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*see_bsPrZco-P6-7YDYsFw.png)

Allow ICMP in Windows firewall on Public and Private VMs

Because these are Windows devices, they will block inbound ICMP Echo packets, by default. We will need this enabled so that we can trace routing during the testing of this solution in our next step. The TRACERT diagnostic utility determines the route to a destination by sending Internet Control Message Protocol (ICMP) echo packets to the destination.Here, we have the option to add an inbound firewall rule to allow ICMP (from the Firewall GUI interface, from PowerShell, or we can disable the Windows firewall temporarily to allow ICMP.

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*xKw_BFvk8S7jeWcl4gCHqg.png)

The NVA has a private IP address of 10.0.2.4

![Image](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*_qI6T4d6Nd5V5Oy5JeDsAg.png)

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*6nEONGPqQuusxmEPgvrlPA.png)

7. Test the routing of network traffic between the Public and Private VMs
Here, I connected to myVMPublic VM, and used PowerShell to execute the tracert command to trace the routing of network traffic from myVMPublic VM to myVMPrivate VM. We see that there are two hops needed in order for traffic to reach the Private VM from the Public VM. The first hop is that of the NVA device with IP address 10.0.2.4. The second hop finally gets us to the private VM at IP address 10.0.1.4.

> tracert nisha-vmprivate

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*KG0fblDTOUKkVo33wXyWqA.png)

In the reverse direction, there is only one hop for traffic from the Private VM to reach the Public VM. This is expected because we did not apply the custom route to the Private-Subnet. Therefore, the default System Routes are still in play for all outbound traffic leaving the Private-Subnet, which allows traffic routing between all subnets in the virtual network, by default.

![Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*awGG6aD0jdlDjrKJcyvzgw.png)

