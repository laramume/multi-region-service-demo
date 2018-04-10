# Multi-region Azure Service

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-jenkins%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## Requirements

1. Regional AppService instances need to access blob storage of regional storage accounts
2. One of the storage accounts is designated as primary; in addition to blobs, this account contains tables used across all regions    
3. The service needs support for atomic and synchronous bulk writes into each regional blob storage (this feature, however, is on a deprecation path in favor of a more simple approach)

## Proof-of-concept design



https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-multiple


https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-multiple#resource-iteration


https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-multiple#property-iteration


https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-multiple#variable-iteration