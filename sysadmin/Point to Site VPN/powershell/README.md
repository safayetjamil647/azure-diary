# Azure Point To Site VPN
## VPN connection for your enterprise

## STEPS

- Step One Create Resources
- Config
- Create VM
- Implement Certificates
- Connect with VPN



## Tech

Dillinger uses a number of open source projects to work properly:

- [Azure CloudShell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview)  Azure CloudShell for interact with Azure
- [Azure Powershell Module](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-8.2.0) - awesome web-based text editor.
- [Azure VNET ](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview) - Markdown parser done right. Fast and easy to extend.
- [Azure VPN Gateway](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways) - great UI boilerplate for modern web apps.
- [Windows VM for Client]

###
Point-to-site native Azure certificate authentication connections use the following items, which you configure in this exercise:

- A RouteBased VPN gateway.
- The public key (.cer file) for a root certificate, which is uploaded to Azure. Once the certificate is uploaded, it is considered a trusted certificate and is used for authentication.
- A client certificate that is generated from the root certificate. The client certificate installed on each client computer that will connect to the VNet. This certificate is used for client authentication.
- VPN client configuration. The VPN client is configured using VPN client configuration files. These files contain the necessary information for the client to connect to the VNet. The files configure the existing VPN client that is native to the operating system. Each client that connects must be configured using the settings in the configuration files.
## Implementation


### Example Values
You can use the following values to create a test environment, or refer to these values to better understand the examples in this article:

VNet

VNet Name: VNet1
Address space: 10.1.0.0/16
For this example, we use only one address space. You can have more than one address space for your VNet.
Subnet name: FrontEnd
Subnet address range: 10.1.0.0/24
Subscription: If you have more than one subscription, verify that you're using the correct one.
Resource Group: TestRG1
Location: East US
Virtual network gateway

Virtual network gateway name: VNet1GW
Gateway type: VPN
VPN type: Route-based
SKU: VpnGw2
Generation: Generation2
Gateway subnet address range: 10.1.255.0/27
Public IP address name: VNet1GWpip
Connection type and client address pool

Connection type: Point-to-site
Client address pool: 172.16.201.0/24
VPN clients that connect to the VNet using this point-to-site connection receive an IP address from the client address pool.


### Declare variables
We use variables for this article so that you can easily change the values to apply to your own environment without having to change the examples themselves. Declare the variables that you want to use. You can use the following sample, substituting the values for your own when necessary. If you close your PowerShell/Cloud Shell session at any point during the exercise, just copy and paste the values again to re-declare the variables.

```
$VNetName  = "VNet1"
$FESubName = "FrontEnd"
$GWSubName = "GatewaySubnet"
$VNetPrefix = "10.1.0.0/16"
$FESubPrefix = "10.1.0.0/24"
$GWSubPrefix = "10.1.255.0/27"
$VPNClientAddressPool = "172.16.201.0/24"
$RG = "TestRG1"
$Location = "EastUS"
$GWName = "VNet1GW"
$GWIPName = "VNet1GWpip"
$GWIPconfName = "gwipconf"
$DNS = "10.2.1.4"
```


Create a VNet
Create a resource group.




```New-AzResourceGroup -Name $RG -Location $Location```

Create the subnet configurations for the virtual network, naming them FrontEnd and GatewaySubnet. These prefixes must be part of the VNet address space that you declared.




```
$fesub = New-AzVirtualNetworkSubnetConfig -Name $FESubName -AddressPrefix $FESubPrefix
$gwsub = New-AzVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix
```
Create the virtual network.

In this example, the -DnsServer server parameter is optional. Specifying a value does not create a new DNS server. The DNS server IP address that you specify should be a DNS server that can resolve the names for the resources you are connecting to from your VNet. This example uses a private IP address, but it is likely that this is not the IP address of your DNS server. Be sure to use your own values. The value you specify is used by the resources that you deploy to the VNet, not by the P2S connection or the VPN client.

Azure PowerShell



Try It
```
    New-AzVirtualNetwork `
   -ResourceGroupName $RG `
   -Location $Location `
   -Name $VNetName `
   -AddressPrefix $VNetPrefix `
   -Subnet $fesub, $gwsub `
   -DnsServer $DNS
   ```
Specify the variables for the virtual network you created.

Azure PowerShell


```
$vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
```

A VPN gateway must have a Public IP address. You first request the IP address resource, and then refer to it when creating your virtual network gateway. The IP address is dynamically assigned to the resource when the VPN gateway is created. VPN Gateway currently only supports Dynamic Public IP address allocation. You cannot request a Static Public IP address assignment. However, it doesn't mean that the IP address changes after it has been assigned to your VPN gateway. The only time the Public IP address changes is when the gateway is deleted and re-created. It doesn't change across resizing, resetting, or other internal maintenance/upgrades of your VPN gateway.

Request a dynamically assigned public IP address.

Azure PowerShell


```
$pip = New-AzPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
$ipconf = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
```