# Plan for IP addressing

1. Don't create unnecessarily large virtual networks (for example, /16) to ensure that IP address space isn't wasted.

```Kusto
resources
| where type == "microsoft.network/virtualnetworks"
| extend addressSpace = todynamic(properties.addressSpace)
| extend addressPrefix = todynamic(properties.addressSpace.addressPrefixes)
| mvexpand addressSpace
| mvexpand addressPrefix
| extend addressMask = split(addressPrefix,'/')[1]
| where addressMask <= 16
``  
