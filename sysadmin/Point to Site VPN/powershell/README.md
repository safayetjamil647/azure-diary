# Azure Point To Site VPN
## VPN connection for your enterprise

## STEPS

- Step One Create Resources
- Config
- Create VM
- Implement Certificates
- Connect with VPN



## Tech

Tech we used for P2S VPN Gateway:

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

### Example Values
You can use the following values to create a test environment, or refer to these values to better understand the examples in this article:

VNet Sample Data

| Name | Value |
| ------ | ------ |
| VNet Name | VNet1 |
| Address space | 10.1.0.0/16 |
For this example, we use only one address space. You can have more than one address space for your VNet.
| Subnet name | FrontEnd |
| Subnet address range | 10.1.0.0/24 |
Subscription: If you have more than one subscription, verify that you're using the correct one.
| Resource Group | TestRG1 |
| Location | East US |

Virtual network gateway

| Name | Value |
| ------ | ------ |
| Virtual network gateway name| VNet1GW |
| Gateway type | VPN |
For this example, we use only one address space. You can have more than one address space for your VNet.
| VPN type | Route-based |
| SKU | VpnGw2 |
| Gateway subnet address range| 10.1.255.0/27|
| Public IP address name | VNet1GWpip |

Connection type and client address pool

Connection type: Point-to-site
Client address pool: 172.16.201.0/24
VPN clients that connect to the VNet using this point-to-site connection receive an IP address from the client address pool.

### Implentation Zone

Create resource group
Create an Azure resource group with New-AzResourceGroup. A resource group is a logical container into which Azure resources are deployed and managed.

```
New-AzResourceGroup -Name 'myResourceGroup' -Location 'EastUS'
```
Create virtual machine
Create a VM with New-AzVM. Provide names for each of the resources and the New-AzVM cmdlet creates if they don't already exist.

When prompted, provide a username and password to be used as the sign-in credentials for the VM:

```
New-AzVm `
    -ResourceGroupName 'myResourceGroup' `
    -Name 'myVM' `
    -Location 'East US' `
    -VirtualNetworkName 'myVnet' `
    -SubnetName 'mySubnet' `
    -SecurityGroupName 'myNetworkSecurityGroup' `
    -PublicIpAddressName 'myPublicIpAddress' `
```


#### Install Az PowerShell

Install the Azure Az PowerShell module


This article explains how to install the Azure Az PowerShell module from The PowerShell Gallery. These instructions work on Windows, Linux, and macOS platforms.

The Azure Az PowerShell module is preinstalled in Azure Cloud Shell and in Docker images.

The Azure Az PowerShell module is a rollup module. Installing it downloads the generally available Az PowerShell modules, and makes their cmdlets available for use.

Requirements
 Note

PowerShell 7.0.6 LTS, PowerShell 7.1.3, or higher is the recommended version of PowerShell for use with the Azure Az PowerShell module on all platforms.

Azure PowerShell has no additional requirements when run on PowerShell 7.0.6 LTS and PowerShell 7.1.3 or higher.

Install the latest version of PowerShell available for your operating system.
To check your PowerShell version, run the following command from within a PowerShell session:

```
$PSVersionTable.PSVersion
```
PowerShell script execution policy must be set to remote signed or less restrictive. Get-ExecutionPolicy -List can be used to determine the current execution policy. For more information, see about_Execution_Policies.

```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
Installation
Using the Install-Module cmdlet is the preferred installation method for the Az PowerShell module. Install the Az module for the current user only. This is the recommended installation scope. This method works the same on Windows, Linux, and macOS platforms. Run the following command from a PowerShell session:
```
Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force
```
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


Create the subnet configurations for the virtual network, naming them FrontEnd and GatewaySubnet. These prefixes must be part of the VNet address space that you declared.




```
$fesub = New-AzVirtualNetworkSubnetConfig -Name $FESubName -AddressPrefix $FESubPrefix
$gwsub = New-AzVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix
```
Create the virtual network.

In this example, the -DnsServer server parameter is optional. Specifying a value does not create a new DNS server. The DNS server IP address that you specify should be a DNS server that can resolve the names for the resources you are connecting to from your VNet. This example uses a private IP address, but it is likely that this is not the IP address of your DNS server. Be sure to use your own values. The value you specify is used by the resources that you deploy to the VNet, not by the P2S connection or the VPN client.


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


```
$vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet

```

A VPN gateway must have a Public IP address. You first request the IP address resource, and then refer to it when creating your virtual network gateway. The IP address is dynamically assigned to the resource when the VPN gateway is created. VPN Gateway currently only supports Dynamic Public IP address allocation. You cannot request a Static Public IP address assignment. However, it doesn't mean that the IP address changes after it has been assigned to your VPN gateway. The only time the Public IP address changes is when the gateway is deleted and re-created. It doesn't change across resizing, resetting, or other internal maintenance/upgrades of your VPN gateway.

Request a dynamically assigned public IP address.


```
$pip = New-AzPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
$ipconf = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
```


Create the VPN gateway
In this step, you configure and create the virtual network gateway for your VNet.

The -GatewayType must be Vpn and the -VpnType must be RouteBased.
The -VpnClientProtocol is used to specify the types of tunnels that you would like to enable. The tunnel options are OpenVPN, SSTP, and IKEv2. You can choose to enable one of them or any supported combination. If you want to enable multiple types, then specify the names separated by a comma. OpenVPN and SSTP cannot be enabled together. The strongSwan client on Android and Linux and the native IKEv2 VPN client on iOS and macOS will use only the IKEv2 tunnel to connect. Windows clients try IKEv2 first and if that doesn’t connect, they fall back to SSTP. You can use the OpenVPN client to connect to OpenVPN tunnel type.
The virtual network gateway 'Basic' SKU does not support IKEv2, OpenVPN, or RADIUS authentication. If you are planning on having Mac clients connect to your virtual network, do not use the Basic SKU.
A VPN gateway can take 45 minutes or more to complete, depending on the gateway sku you select.
Configure and create the virtual network gateway for your VNet. It takes approximately 45 minutes for the gateway to create.
```
New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
-Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
-VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1 -VpnClientProtocol "IKEv2"
```
Once your gateway is created, you can view it using the following example. If you closed PowerShell or it timed out while your gateway was being created, you can declare your variables again.

```
Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroup $RG
```
Add the VPN client address pool
After the VPN gateway finishes creating, you can add the VPN client address pool. The VPN client address pool is the range from which the VPN clients receive an IP address when connecting. Use a private IP address range that does not overlap with the on-premises location that you connect from, or with the VNet that you want to connect to.

In this example, the VPN client address pool is declared as a variable in an earlier step.

```
$Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway -VpnClientAddressPool $VPNClientAddressPool
```


You can't generate certificates using Azure Cloud Shell. You must use one of the methods outlined in this section. If you want to use PowerShell, you must install it locally.

Certificates are used by Azure to authenticate VPN clients for point-to-site VPNs. You upload the public key information of the root certificate to Azure. The public key is then considered 'trusted'. Client certificates must be generated from the trusted root certificate, and then installed on each client computer in the Certificates-Current User/Personal certificate store. The certificate is used to authenticate the client when it initiates a connection to the VNet.

If you use self-signed certificates, they must be created using specific parameters. You can create a self-signed certificate using the instructions for PowerShell and Windows 10 or later, or, if you don't have Windows 10 or later, you can use MakeCert. It's important that you follow the steps in the instructions when generating self-signed root certificates and client certificates. Otherwise, the certificates you generate will not be compatible with P2S connections and you receive a connection error.

Self-signed root certificate: If you aren't using an enterprise certificate solution, create a self-signed root certificate. Otherwise, the certificates you create won't be compatible with your P2S connections and clients will receive a connection error when they try to connect. You can use Azure PowerShell, MakeCert, or OpenSSL. The steps in the following articles describe how to generate a compatible self-signed root certificate:

Windows 10 or later PowerShell instructions: These instructions require Windows 10 or later and PowerShell to generate certificates. Client certificates that are generated from the root certificate can be installed on any supported P2S client.
MakeCert instructions: Use MakeCert if you don't have access to a Windows 10 or later computer to use to generate certificates. Although MakeCert is deprecated, you can still use it to generate certificates. Client certificates that you generate from the root certificate can be installed on any supported P2S client.
Linux instructions.
After you create the root certificate, export the public certificate data (not the private key) as a Base64 encoded X.509 .cer file.

Client certificate
Each client computer that you connect to a VNet with a Point-to-Site connection must have a client certificate installed. You generate it from the root certificate and install it on each client computer. If you don't install a valid client certificate, authentication will fail when the client tries to connect to the VNet.

You can either generate a unique certificate for each client, or you can use the same certificate for multiple clients. The advantage to generating unique client certificates is the ability to revoke a single certificate. Otherwise, if multiple clients use the same client certificate to authenticate and you revoke it, you'll need to generate and install new certificates for every client that uses that certificate.

You can generate client certificates by using the following methods:


Self-signed root certificate: Follow the steps in one of the following P2S certificate articles so that the client certificates you create will be compatible with your P2S connections.

When you generate a client certificate from a self-signed root certificate, it's automatically installed on the computer that you used to generate it. If you want to install a client certificate on another client computer, export it as a .pfx file, along with the entire certificate chain. Doing so will create a .pfx file that contains the root certificate information required for the client to authenticate.

The steps in these articles generate a compatible client certificate, which you can then export and distribute.

Windows 10 or later PowerShell instructions: These instructions require Windows 10 or later, and PowerShell to generate certificates. The generated certificates can be installed on any supported P2S client.

Create a self-signed root certificate

Use the New-SelfSignedCertificate cmdlet to create a self-signed root certificate. For additional parameter information, see New-SelfSignedCertificate.

From a computer running Windows 10 or later, or Windows Server 2016, open a Windows PowerShell console with elevated privileges. These examples don't work in the Azure Cloud Shell "Try It". You must run these examples locally.

Use the following example to create the self-signed root certificate. The following example creates a self-signed root certificate named 'P2SRootCert' that is automatically installed in 'Certificates-Current User\Personal\Certificates'. You can view the certificate by opening certmgr.msc, or Manage User Certificates.

Run the following example with any necessary modifications.

PowerShell

Copy
```
$cert = New-SelfSignedCertificate -Type Custom -KeySpec Signature `
-Subject "CN=P2SRootCert" -KeyExportPolicy Exportable `
-HashAlgorithm sha256 -KeyLength 2048 `
-CertStoreLocation "Cert:\CurrentUser\My" -KeyUsageProperty Sign -KeyUsage CertSign
```
Leave the PowerShell console open and proceed with the next steps to generate a client certificate.

Generate a client certificate
Each client computer that connects to a VNet using Point-to-Site must have a client certificate installed. You generate a client certificate from the self-signed root certificate, and then export and install the client certificate. If the client certificate isn't installed, authentication fails.

The following steps walk you through generating a client certificate from a self-signed root certificate. You may generate multiple client certificates from the same root certificate. When you generate client certificates using the steps below, the client certificate is automatically installed on the computer that you used to generate the certificate. If you want to install a client certificate on another client computer, you can export the certificate.

The examples use the New-SelfSignedCertificate cmdlet to generate a client certificate that expires in one year. For additional parameter information, such as setting a different expiration value for the client certificate, see New-SelfSignedCertificate.

Example 1 - PowerShell console session still open
Use this example if you haven't closed your PowerShell console after creating the self-signed root certificate. This example continues from the previous section and uses the declared '$cert' variable. If you closed the PowerShell console after creating the self-signed root certificate, or are creating additional client certificates in a new PowerShell console session, use the steps in Example 2.

Modify and run the example to generate a client certificate. If you run the following example without modifying it, the result is a client certificate named 'P2SChildCert'. If you want to name the child certificate something else, modify the CN value. Don't change the TextExtension when running this example. The client certificate that you generate is automatically installed in 'Certificates - Current User\Personal\Certificates' on your computer.
```
New-SelfSignedCertificate -Type Custom -DnsName P2SChildCert -KeySpec Signature `
-Subject "CN=P2SChildCert" -KeyExportPolicy Exportable `
-HashAlgorithm sha256 -KeyLength 2048 `
-CertStoreLocation "Cert:\CurrentUser\My" `
-Signer $cert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")
```

Upload root certificate public key information
Verify that your VPN gateway has finished creating. Once it has completed, you can upload the .cer file (which contains the public key information) for a trusted root certificate to Azure. Once a .cer file is uploaded, Azure can use it to authenticate clients that have installed a client certificate generated from the trusted root certificate. You can upload additional trusted root certificate files - up to a total of 20 - later, if needed.

 Note

You can't upload the .cer file using Azure Cloud Shell. You can either use PowerShell locally on your computer, or you can use the Azure portal steps.

Declare the variable for your certificate name, replacing the value with your own.


```
$P2SRootCertName = "P2SRootCert.cer"
```


```
$filePathForCert = "C:\cert\P2SRootCert.cer"
$cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
$CertBase64 = [system.convert]::ToBase64String($cert.RawData)
```
Upload the public key information to Azure. Once the certificate information is uploaded, Azure considers it to be a trusted root certificate. When uploading, make sure you are running PowerShell locally on your computer, or instead, you can use the Azure portal steps. You can't upload using Azure Cloud Shell.

```
Add-AzVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG1" -PublicCertData $CertBase64
```

Install an exported client certificate
The following steps help you install on a Windows client. For additional clients and more information, see Install a client certificate.

Once the client certificate is exported, locate and copy the .pfx file to the client computer.
On the client computer, double-click the .pfx file to install. Leave the Store Location as Current User, and then select Next.
On the File to import page, don't make any changes. Select Next.
On the Private key protection page, input the password for the certificate, or verify that the security principal is correct, then select Next.
On the Certificate Store page, leave the default location, and then select Next.
Select Finish. On the Security Warning for the certificate installation, select Yes. You can comfortably select 'Yes' for this security warning because you generated the certificate.
The certificate is now successfully imported.
Make sure the client certificate was exported as a .pfx along with the entire certificate chain (which is the default). Otherwise, the root certificate information isn't present on the client computer and the client won't be able to authenticate properly.

Configure the VPN client
To connect to the virtual network gateway using P2S, each computer uses the VPN client that is natively installed as a part of the operating system. For example, when you go to VPN settings on your Windows computer, you can add VPN connections without installing a separate VPN client. You configure each VPN client by using a client configuration package. The client configuration package contains settings that are specific to the VPN gateway that you created.

You can use the following quick examples to generate and install the client configuration package. For more information about package contents and additional instructions about to generate and install VPN client configuration files, see Create and install VPN client configuration files.

If you need to declare your variables again, you can find them here.

To generate configuration files
```
$profile=New-AzVpnClientConfiguration -ResourceGroupName $RG -Name $GWName -AuthenticationMethod "EapTls"
$profile.VPNProfileSASUrl
```
To install the client configuration package
You can use the same VPN client configuration package on each Windows client computer, as long as the version matches the architecture for the client. For the list of client operating systems that are supported, see the Point-to-Site section of the VPN Gateway FAQ.

 Note

You must have Administrator rights on the Windows client computer from which you want to connect.

### Install the configuration files
Select the VPN client configuration files that correspond to the architecture of the Windows computer. For a 64-bit processor architecture, choose the 'VpnClientSetupAmd64' installer package. For a 32-bit processor architecture, choose the 'VpnClientSetupX86' installer package.
Double-click the package to install it. If you see a SmartScreen popup, click More info, then Run anyway.
Verify and connect
Verify that you have installed a client certificate on the client computer. A client certificate is required for authentication when using the native Azure certificate authentication type. To view the client certificate, open Manage User Certificates. The client certificate is installed in Current User\Personal\Certificates.
To connect, navigate to Network Settings and click VPN. The VPN connection shows the name of the virtual network that it connects to.
 Connect to Azure
Windows VPN client


You must have Administrator rights on the Windows client computer from which you are connecting.

To connect to your VNet, on the client computer, navigate to VPN settings and locate the VPN connection that you created. It's named the same name as your virtual network. Select Connect. A pop-up message may appear that refers to using the certificate. Select Continue to use elevated privileges.

On the Connection status page, select Connect to start the connection. If you see a Select Certificate screen, verify that the client certificate showing is the one that you want to use to connect. If it is not, use the drop-down arrow to select the correct certificate, and then select OK.

