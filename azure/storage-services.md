Azure Storage Services:

 - Storage account: An Azure Storage Account is a container that holds all your Azure Storage data objects, including blobs, files, queues, and tables

Can store:
   1. Blob container
   2. File
   3. Tables
   4. Queues

Blob storage:
 - Can store any kind of unstructured data images, videos, etc

File storage:
  - When you want to share the files between multiple VMs or multiple pods.

Tables:
  - Store NoSQL data, store key-value pair
  - Suppose when you want to store some customer information in tables
  - Its schema less unlike database


Queue:
  - Its like SQS in AWS
  - Provides Queue service
  - When an application wants to process many request, it can use Queue storage and process it one by one.

 
- We can host static website in storage account

Types:
 1. Standard general-purpose v2
 2. Premium block blobs3
 3. Premium file shares3
 4. Premium page blobs3

- Securing storage account:
    - Disabled public access
    - Enable encryption
    - Use private endpoint for access
    - Use Microsoft Entra ID/RBAC for access
    - Enable logging and monitoring
 
Replication:
  - Azure Storage provides different redundancy options.
| Replication | Concept                                                                 |
| ----------- | ----------------------------------------------------------------------- |
| LRS.        | Copies data within a single physical location/datacenter scope          |
| ZRS         | Replicates synchronously across availability zones in the region        |
| GRS         | Replicates to a secondary region                                        |
| GZRS        | Zone redundancy in primary region + geo-replication to secondary region |


Access:
 1. SAS: A shared access signature (SAS) is a URI that grants restricted access rights to Azure Storage resources. You can provide a shared access signature to clients who should not be trusted with your storage account key but whom you wish to delegate access to certain storage account resources. By distributing a shared access signature URI to these clients, you grant them access to a resource for a specified period of time.

 2. Access keys: Storage Account access keys are shared secret credentials used to authenticate access to Azure Storage.

 3. Microsoft Entra ID: Use workload identity for Pods to access the Storage account.
 
  
