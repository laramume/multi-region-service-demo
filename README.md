# Multi-region Azure Service

## Problem statement

- Regional AppService instances need to have access to blob storage of regional storage accounts
- One of the storage accounts is designated as primary; in addition to blobs, this account contains tables used across all regions    
- The service needs support for atomic and synchronous bulk writes into each regional blob storage (this feature, however, is on a deprecation path in favor of a more simple approach)

## Proof of concept

