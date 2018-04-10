# Multi-region Azure Service via ARM

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fwdecay%2Fmulti-region-service-demo%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## Overview

This document outlines one possible solution to the problem of deploying a multi-region Azure service backed by Azure Storage. The service consists of regional App Service instances connected to both primary and regional storage accounts. The primary storage is shared accross all the regions, whereas regional storage, containing relatively static blobs, is mirrored in each region in order to reduce network latencies.

## Requirements

1. Regional App Service instances need read-only access to blob storage of regional storage accounts.
2. One of the storage accounts is designated as primary; in addition to blobs, this account contains tables used across all the regions.
3. The service needs to support atomic and synchronous bulk writes into each regional blob storage (this feature, however, is on a deprecation path in favor of a more simple approach).

## High-level design

To satisfy reqirements 1, 2 and 3, each regional App Service needs to, respectively:

1. be aware of its assigned regional storage
2. be aware of the designated primary storage
3. be aware of all existing storage accounts.

This can be achieved by making Connection Strings and App Settings work together.

### Making Connection Strings and App Settings work together

Let all regional App Services contain a full set of connection strings pointing to every regional storage account with names of the Connection Strings being in the form of `storage-<location>`.

Given that Connection Strings pointing to all storage accounts are now available, regional and primary storages for each App Service can be designated by adding two indexer App Settings &mdash; one containing the name of the primary connection string, and another containing the name of the regional connection string. For example:

```json
{
    "PRIMARY_STORAGE_CONNECTION_STRING": "storage-eastus",
    "REGIONAL_STORAGE_CONNECTION_STRING": "storage-westus"
}
```

Resolving connection strings for both storage accounts within the service is now trivial:

```python
primary_storage = connection_strings[app_settings['PRIMARY_STORAGE_CONNECTION_STRING']]
regional_storage = connection_strings[app_settings['REGIONAL_STORAGE_CONNECTION_STRING']]
```

If simultaneous access to all storage accounts is needed, it's a matter of reading all Connection Strings.

### Example

Suppose, the service is deployed to 3 Azure regions: East US, North Europe and West US.

![](img/resources.png)

If East US is the location of the primary storage, connection Strings and App Settings of the App Service deployed in North Europe would look like this:

![](img/settings.png)

## Proof-of-concept implementation

To learn more about the resource iteration techniques employed in the [example ARM template](azuredeploy.json), see [Deploy multiple instances of a resource or property in Azure Resource Manager templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-multiple). These techniques include:


- [Resource iteration](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-multiple#resource-iteration)
     (creating regional instances of various resources)


- [Property iteration](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-multiple#property-iteration)
    (creating a variable number of Connection Strings)


- [Variable iteration](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-multiple#variable-iteration) 
    (generating an array of all regional resources along with their attributes)

- [Depend on resources in a loop](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-multiple#depend-on-resources-in-a-loop)

The template expects one parameter, which is an array of valid Azure regions. By convention, the primary storage account is the one created in the region that appears first in that array.

## Conclusion

The proposed solution satisfies all the requirements. As mentioned earlier, the feature requiring simultaneous write access to all regional storage accounts is on a deprecation path. There are, however, other benefits from giving the service read acccess to all available regional storage accounts. In particular, that can be leveraged to enable a failover when a regional storage account becomes unavailable. It can also enable scenarios where the service deployed to region A needs to perform work in region B (e.g., a user in North America, needs to copy blobs between accounts in Australia).