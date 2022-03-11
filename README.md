# Azure Resource Graph Queries
A collection on Azure Resource Graph Queries that I use fairly regularly. Working to have the queries listed in different folders in an attempt to make them easier to find.

These queries are designed to be run in [Azure Resource Graph](https://docs.microsoft.com/en-us/azure/governance/resource-graph/overview) and **NOT** in Azure Monitor, so some the functionality would be different. Not all aspects of the query language work in Azure Resource Graph.

[Azure Resource Graph Explorer Portal Link](https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade)

[Understanding the Azure Resource Graph query language](https://docs.microsoft.com/en-us/azure/governance/resource-graph/concepts/query-language)

## Understanding the symbols

Getting these queries correct was actually started by an internal [FastTrack for Azure](https://aks.ms/fta) project called [Azure Review Checklists](https://github.com/Azure/review-checklists) and started with a query for compliant and non-compliant. Over time the approach has changed to use one query and check for compliant and non-compliant items. I will continue to attempt to build all of teh types of queries for learning and so that other people can easily adapt them.

✅ Single check for **COMPLIANT** items  
❌ Single check for **NON COMPLIANT** items  
✅❌ Single check that check **COMPLAINT** and **NON COMPLIANT** items.  

## Running the queries

THe queries are designed to be a simple copy paste exercise. Copy the query and paste it into [Azure Resource Graph Explorer Portal Link](https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade) and hit _Run Query_

## Where are the queries?

- [Keyvault queries](/keyVaults//keyvault.md)
- [Management Group queries](/managementGroups//managementGroups.md)
- [Networking queries](/networking//networking.md)