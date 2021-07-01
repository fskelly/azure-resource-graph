# Queries

1. Configure a default, dedicated management group for new subscriptions to ensure that no subscriptions are placed under the root management group. This is especially important if there are users eligible for MSDN or Visual Studio benefits and subscriptions. A good candidate for this type of management group is a Sandbox management group.

```kusto
resourcecontainers
| where type == "microsoft.resources/subscriptions"
| extend ManagementGroup = tostring(tags),mgmtChain = properties.managementGroupAncestorsChain
| where array_length(mgmtChain) ==1
|mvexpand mgmt=mgmtChain
|project id, subName = name, subscriptionId, managedBy, managementGroupName = mgmt.displayName
```

1. Keep the management group hierarchy reasonably flat, with no more than three to four levels ideally. This restriction reduces management overhead and complexity.

```kusto
resourcecontainers
| where type == "microsoft.resources/subscriptions"
| extend ManagementGroup = tostring(tags),mgmtChain = properties.managementGroupAncestorsChain
| where array_length(mgmtChain) >=4
|mvexpand mgmt=mgmtChain
