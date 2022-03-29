# Queries

**Don't create unnecessarily large virtual networks (for example, /16) to ensure that IP address space isn't wasted.**

✅ Recommended sized Networks

```kusto
resources 
| where type == 'microsoft.network/virtualnetworks' 
| extend addressSpace = todynamic(properties.addressSpace) 
| extend addressPrefix = todynamic(properties.addressSpace.addressPrefixes) 
| mvexpand addressSpace 
| mvexpand addressPrefix 
| extend addressMask = split(addressPrefix,'/')[1] 
| where addressMask > 16 | project name, id, subscriptionId, resourceGroup, addressPrefix
```

❌ Large networks

```kusto
resources 
| where type == 'microsoft.network/virtualnetworks' 
| extend addressSpace = todynamic(properties.addressSpace) 
| extend addressPrefix = todynamic(properties.addressSpace.addressPrefixes) 
| mvexpand addressSpace 
| mvexpand addressPrefix 
| extend addressMask = split(addressPrefix,'/')[1] 
| where addressMask <= 16 
| project name, id, subscriptionId, resourceGroup, addressPrefix
```

✅❌ Combined Check

```kusto
resources 
| where type == 'microsoft.network/virtualnetworks' 
| extend addressSpace = todynamic(properties.addressSpace) 
| extend addressPrefix = todynamic(properties.addressSpace.addressPrefixes) 
| mvexpand addressSpace 
| mvexpand addressPrefix 
| extend addressMask = split(addressPrefix,'/')[1] 
| extend Compliant = addressMask >= 16 
| project name, id, subscriptionId, resourceGroup, addressPrefix, Compliant
```

Smallest recommended size for a GatewaySubnet is /27

**When you are planning your gateway subnet size, refer to the documentation for the configuration that you are planning to create. For example, the ExpressRoute/VPN Gateway coexist configuration requires a larger gateway subnet than most other configurations. Additionally, you may want to make sure your gateway subnet contains enough IP addresses to accommodate possible future additional configurations. While you can create a gateway subnet as small as /29, we recommend that you create a gateway subnet of /27 or larger (/27, /26 etc.) if you have the available address space to do so. This will accommodate most configurations.** [Microsoft Docs](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings#gwsub)

❌ Gateway too small

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

**Use IP addresses from the address allocation for private internets (RFC 1918).** [Plan for IP addressing](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/plan-for-ip-addressing)

✅ RFC 1918 Compliant

```kusto
resources 
| where type == 'microsoft.network/virtualnetworks' 
| extend addressSpace = todynamic(properties.addressSpace) 
| extend addressPrefix = todynamic(properties.addressSpace.addressPrefixes) 
| mvexpand addressSpace 
| mvexpand addressPrefix 
| project name, id, location, resourceGroup, subscriptionId, cidr = addressPrefix 
| where (cidr matches regex @'^(10(\.(25[0-5]|2[0-4][0-9]|1[0-9]{1,2}|[0-9]{1,2})){3}|((172\.(1[6-9]|2[0-9]|3[01]))|192\.168)(\.(25[0-5]|2[0-4][0-9]|1[0-9]{1,2}|[0-9]{1,2})){2})')  | project name, id, subscriptionId, resourceGroup,cidr
```

❌ **NOT** RFC 1918 Compliant

```kusto
resources 
| where type == 'microsoft.network/virtualnetworks' 
| extend addressSpace = todynamic(properties.addressSpace) 
| extend addressPrefix = todynamic(properties.addressSpace.addressPrefixes) 
| mvexpand addressSpace 
| mvexpand addressPrefix 
| project name, id, location, resourceGroup, subscriptionId, cidr = addressPrefix 
| where not (cidr matches regex @'^(10(\.(25[0-5]|2[0-4][0-9]|1[0-9]{1,2}|[0-9]{1,2})){3}|((172\.(1[6-9]|2[0-9]|3[01]))|192\.168)(\.(25[0-5]|2[0-4][0-9]|1[0-9]{1,2}|[0-9]{1,2})){2})')  | project name, id, subscriptionId, resourceGroup,cidr
```

✅❌ Combined Check

```kusto
resources 
| where type == 'microsoft.network/virtualnetworks' 
| extend addressSpace = todynamic(properties.addressSpace) 
| extend addressPrefix = todynamic(properties.addressSpace.addressPrefixes) 
| mvexpand addressSpace 
| mvexpand addressPrefix 
| project name, id, location, resourceGroup, subscriptionId, cidr = addressPrefix 
| extend Compliant = (cidr matches regex @'^(10(\.(25[0-5]|2[0-4][0-9]|1[0-9]{1,2}|[0-9]{1,2})){3}|((172\.(1[6-9]|2[0-9]|3[01]))|192\.168)(\.(25[0-5]|2[0-4][0-9]|1[0-9]{1,2}|[0-9]{1,2})){2})')  | project name, id, subscriptionId, resourceGroup,Compliant,cidr
```

✅ VPN gateway not to use basic in production

```kusto
resources
| where type == "microsoft.network/virtualnetworkgateways"
| where properties.gatewayType =~ "vpn" or properties.gatewayType == "ExpressRoute"
| extend SKUName = properties.sku.name, SKUTier = properties.sku.tier, Type = properties.gatewayType
| where SKUTier != "Basic" and SKUTier != "Standard"
```

❌ VPN gateway not to use basic in production

```kusto
resources
| where type == "microsoft.network/virtualnetworkgateways"
| where properties.gatewayType =~ "vpn" or properties.gatewayType == "ExpressRoute"
| extend SKUName = properties.sku.name, SKUTier = properties.sku.tier, Type = properties.gatewayType
| where SKUTier == "Basic" or SKUTier == "Standard"
```

✅❌ Combined Check

```kusto
resources
| where type == "microsoft.network/virtualnetworkgateways"
| where properties.gatewayType =~ "vpn" or properties.gatewayType == "ExpressRoute"
| extend SKUName = properties.sku.name, SKUTier = properties.sku.tier, Type = properties.gatewayType
| extend Compliant = SKUTier != "Basic" and SKUTier != "Standard"
| project name, id, subscriptionId, resourceGroup, Compliant
```

✅ Gateway deployed in Availability Zone

```kusto
resources
| where type == "microsoft.network/virtualnetworkgateways"
| where properties.gatewayType =~ "vpn" or properties.gatewayType == "ExpressRoute"
| extend SKUName = properties.sku.name, SKUTier = properties.sku.tier, Type = properties.gatewayType
| where SKUTier contains "AZ"
| project name, id, subscriptionId, resourceGroup, Type
```

❌ Gateway deployed in Availability Zone

```kusto
resources
| where type == "microsoft.network/virtualnetworkgateways"
| where properties.gatewayType =~ "vpn" or properties.gatewayType == "ExpressRoute"
| extend SKUName = properties.sku.name, SKUTier = properties.sku.tier, Type = properties.gatewayType
| where SKUTier notcontains "AZ"
| project name, id, subscriptionId, resourceGroup, Type
```

✅❌ Combined Check

```kusto
resources
| where type == "microsoft.network/virtualnetworkgateways"
| where properties.gatewayType =~ "vpn" or properties.gatewayType == "ExpressRoute"
| extend SKUName = properties.sku.name, SKUTier = properties.sku.tier, Type = properties.gatewayType
| extend Compliant = SKUTier contains "AZ"
| project name, id, subscriptionId, resourceGroup, Type, Compliant
```
