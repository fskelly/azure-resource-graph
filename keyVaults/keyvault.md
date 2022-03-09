# Queries

Use different Azure Key Vaults for different applications and regions to avoid transaction scale limits and restrict access to secrets.

✅No Keyvault or multiple keyvaults found

```kusto
ResourceContainers 
| where type=='microsoft.resources/subscriptions'
| parse id with '/subscriptions/' SubscriptionID
| project subscriptionId, SubscriptionName = name
| join kind=leftouter (Resources
| where type == 'microsoft.keyvault/vaults'
| project id, name, subscriptionId) on subscriptionId
| join kind= leftouter (Resources| where type == 'microsoft.keyvault/vaults'
| summarize ResourceCount = count() by subscriptionId) on subscriptionId
| extend RCount = iff(isnull(ResourceCount), 0, ResourceCount)
| project-away ResourceCount| where RCount <> 1
```

❌ Only 1 Keyvault found

```kusto
ResourceContainers 
| where type=='microsoft.resources/subscriptions'
| parse id with '/subscriptions/' SubscriptionID
| project subscriptionId, SubscriptionName = name
| join kind=leftouter (Resources
| where type == 'microsoft.keyvault/vaults'
| project id, name, subscriptionId) on subscriptionId
| join kind= leftouter (Resources
| where type == 'microsoft.keyvault/vaults'
| summarize ResourceCount = count() by subscriptionId) on subscriptionId
| extend RCount = iff(isnull(ResourceCount), 0, ResourceCount)
| project-away ResourceCount
| where RCount == 1
```
