# Queries

Critical Design Areas -> Networking Toplogy and Connectivity -> [Plan for IP Addressing](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/plan-for-ip-addressing)

**Don't create unnecessarily large virtual networks (for example, /16) to ensure that IP address space isn't wasted.**

```Kusto
resources
| where type == "microsoft.network/virtualnetworks"
| extend addressSpace = todynamic(properties.addressSpace)
| extend addressPrefix = todynamic(properties.addressSpace.addressPrefixes)
| mvexpand addressSpace
| mvexpand addressPrefix
| extend addressMask = split(addressPrefix,'/')[1]
| where addressMask <= 16
```

Smallest recommended size for a GatewaySubnet is /27

**When you are planning your gateway subnet size, refer to the documentation for the configuration that you are planning to create. For example, the ExpressRoute/VPN Gateway coexist configuration requires a larger gateway subnet than most other configurations. Additionally, you may want to make sure your gateway subnet contains enough IP addresses to accommodate possible future additional configurations. While you can create a gateway subnet as small as /29, we recommend that you create a gateway subnet of /27 or larger (/27, /26 etc.) if you have the available address space to do so. This will accommodate most configurations.** [Microsoft Docs](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings#gwsub)

```kusto
resources
| where type == "microsoft.network/virtualnetworks"
| extend subnets = todynamic(properties.subnets)
| mvexpand subnets
| project VNetname=name,SubnetName=subnets.name, resourceGroup, location, CIDR=subnets.properties.addressPrefix, id
| where SubnetName == 'GatewaySubnet'
| extend subnetMask = split(CIDR,'/')[1]
| where subnetMask >= 27
```
