# Storage-Security-Access-Control

Implemente Azure Storage security controls for DMP Consulting by configuring storage firewalls, SAS tokens, stored access policies, access keys, and RBAC. This project focused on securing storage access using Azure best practices and the principle of least privilege.

---
**Scenario**
DMP Consulting stores client documents, contracts, and project deliverables in Azure Storage.

To improve security, leadership requires:

Restricting public access to storage accounts
Providing temporary access for external vendors
Implementing RBAC-based access for internal employees
Understanding how access keys and SAS tokens work
Following the principle of least privilege

As the Azure Administrator, you will build and secure a storage solution while testing multiple access methods.

---
Lab Architecture

[Insert Photo]

---
**Create the Storage Account**

Well first start of by creating a Resource Group
DMP-Storage-RG
Central US

[insert photo]

Then well create a storage account

az group create -n Storage-RG -l centralus
<img width="902" height="220" alt="image" src="https://github.com/user-attachments/assets/cfbf092a-8fc9-4acb-a52a-d11ec3c31b33" />

<img width="1126" height="526" alt="image" src="https://github.com/user-attachments/assets/097dd623-6aaf-40bc-8c98-d6e665be2837" />

Next well create a storage account

az storage account create --name dmpsecurestorage --resource-group Storage-RG  --location centralus  --sku Standard_LRS  --kind StorageV2 
<img width="1122" height="587" alt="image" src="https://github.com/user-attachments/assets/08878962-4013-409d-b155-261672deb909" />

<img width="1110" height="547" alt="image" src="https://github.com/user-attachments/assets/37575af3-86d5-428d-9132-4607f17f04ba" />


LRS: Keeps 3 copies of your data in a single datacenter (one building) within one region.
ZRS: Keeps 3 copies of your data across 3 different availability zones (separate buildings) within the same region.
GRS: Combines LRS with distance. It keeps 3 copies in your primary region, then asynchronously copies them to a secondary region hundreds of miles away.
RA-GRS: Exactly like GRS, but it gives you read-access to the secondary region at all times.

---
**Create Blob Storage**
Now well go ahead and create a container

In Azure Storage, a **container** acts exactly like a folder in the cloud. It is a logical bucket used to organize and store unstructured data called Blobs (Binary Large Objects), such as files, images, videos, or logs.

This is the command we will use to create the storage container
az storage container create --name client-documents --account-name dmpsecurestorage --auth-mode login
<img width="1007" height="67" alt="image" src="https://github.com/user-attachments/assets/db2add3d-50c5-4305-8cd3-cc5602e900a3" />

I will now use a LLM to generate some mock documents
<img width="687" height="302" alt="image" src="https://github.com/user-attachments/assets/37758458-1e73-4ab5-997a-2002e18ca46d" />

These will serve as sample files.

Lets now upload these files.

<img width="1132" height="762" alt="image" src="https://github.com/user-attachments/assets/95954c90-29fd-42bd-98a4-47312183e59c" />

---
Configure Storage Firewall
Lets restrict who can access the storage account.

Currently is enabled from all networks
<img width="1132" height="767" alt="image" src="https://github.com/user-attachments/assets/e69585e1-0b96-44bf-9c34-123b07e8b2e9" />

Lets enabled from selected virtual networks and IP addresses. Lets add my workstation IP and Block everything else

az storage account network-rule add --account-name dmpsecurestorage --resource-group Storage-RG --ip-address "<IP>"
<img width="1136" height="657" alt="image" src="https://github.com/user-attachments/assets/53e7b03d-7de2-4505-bd55-1d8486cc1c67" />

---
**Create a Shared Access Signature (SAS)**
Scenario
An external vendor needs temporary read access.

A Shared Access Signature (SAS) is a secure token that grants limited, temporary access to cloud resources (like files, databases, or storage blobs) without exposing your underlying account keys. It is most commonly used in Microsoft Azure to safely share data with third-party clients.

az storage container generate-sas --account-name dmpsecurestorage --name client-documents --permissions r --expiry 2026-06-25T23:59:59Z --auth-mode login --as-user --https-only --output tsv
<img width="1137" height="72" alt="image" src="https://github.com/user-attachments/assets/a90d6a9b-c5a4-429a-993a-f0cd45cacb57" />

provides read-only access without sharing account keys.

---
**Configure Stored Access Policy**
The goal is to manage SAS permissions centrally.

Currently there are no access policies in place
<img width="1129" height="627" alt="image" src="https://github.com/user-attachments/assets/8e93295e-bb1c-4778-b4c9-03a9e525418d" />

az storage container policy create --account-name dmpsecurestorage --account-key $ACCOUNT_KEY --container-name client-documents --name VendorReadAccess --start 2026-06-20T12:23Z --expiry 2026-06-25T23:59:59Z --permissions r
<img width="1130" height="220" alt="image" src="https://github.com/user-attachments/assets/267560a8-2930-45f2-be8b-c106f1ad840a" />

Newly create policy
<img width="1129" height="587" alt="image" src="https://github.com/user-attachments/assets/647dcaec-8681-48e1-848f-e1fd0d06067e" />

I went ahead and generates a new SAS and linked it to the newly create policy, without stored access policies each SAS must be managed individually. Stored Access Policies allow centralized management.

---
**Manage Access Keys**

Key rotation is the periodic replacement of cryptographic keys to limit the data exposed by a single breach and reduce credential lifespans.

<img width="1132" height="642" alt="image" src="https://github.com/user-attachments/assets/9813482f-4d2f-447a-9903-307a7670740a" />


Microsoft recommends RBAC or SAS because standard account keys grant permanent, unrestricted root access.

RBAC: Replaces static keys with centralized, identity-based permissions that enforce least privilege.
SAS: Restricts access to specific resources using short-lived, self-expiring tokens.

---
**Configure RBAC Access**
Lets go and configure RBAC Access to provide identity-based access

az role assignment create --assignee "Storagereader@devonphillips211gmail.onmicrosoft.com" --role "Storage Blob Data Reader" --scope "/subscriptions/f92d2dac-adc4-4d2d-96c0-b0fa0c6bbb25/resourceGroups/Storage-RG/providers/Microsoft.Storage/storageAccounts/dmpsecurestorage"
<img width="1126" height="435" alt="image" src="https://github.com/user-attachments/assets/56a61590-d170-4816-9a93-2a1909794568" />

Now when we look at the role assignments we see "storage reader"
<img width="1131" height="640" alt="image" src="https://github.com/user-attachments/assets/6a5ab69d-aead-4957-b3e9-026ac4279f04" />

This user can: View blobs and download blobs but does not have access to upload or delete.
<img width="1131" height="677" alt="image" src="https://github.com/user-attachments/assets/92bc97f4-379b-407e-bde9-70654d19d859" />

---
**Comparing Access Methods**
heres a chart ive build thats easy to understand

| Method               | Best Use                    |
| -------------------- | --------------------------- |
| Access Keys          | Legacy / Application Access |
| SAS                  | Temporary Delegated Access  |
| Stored Access Policy | Central SAS Management      |
| RBAC                 | User-Based Access Control   |

---

The goal of this excercise was to:
Configure access to storage 
Configure Azure Storage firewalls and virtual networks
Create and use SAS tokens
Configure stored access policies
Manage access keys
Configure identity-based access for Azure Storage

:)
