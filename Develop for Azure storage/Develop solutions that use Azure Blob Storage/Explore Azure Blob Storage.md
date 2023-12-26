# Explore Azure Blob storage
Azure Blob storage is Microsoft's object storage solution for the cloud. Blob storage is optimized for storing massive amounts of unstructured data. Unstructured data is data that doesn't adhere to a particular data model or definition, such as text or binary data.

Blob storage is designed for:

- Serving images or documents directly to a browser.
- Storing files for distributed access.
- Streaming video and audio.
- Writing to log files.
- Storing data for backup and restore, disaster recovery, and archiving.
- Storing data for analysis by an on-premises or Azure-hosted service.
Users or client applications can access objects in Blob storage via HTTP/HTTPS, from anywhere in the world. Objects in Blob storage are accessible via the Azure Storage REST API, Azure PowerShell, Azure CLI, or an Azure Storage client library. 
An Azure Storage account is the top-level container for all of your Azure Blob storage. The storage account provides a unique namespace for your Azure Storage data that is accessible from anywhere in the world over HTTP or HTTPS.

# Types of storage accounts
Azure Storage offers two performance levels of storage accounts, standard and premium. Each performance level supports different features and has its own pricing model.

- **Standard:** This is the standard general-purpose v2 account and is recommended for most scenarios using Azure Storage.
- **Premium:** Premium accounts offer higher performance by using solid-state drives. If you create a premium account you can choose between three account types, block blobs, page blobs, or file shares.
The following table describes the types of storage accounts recommended by Microsoft for most scenarios using Blob storage.

|Storage account type|Performance tier|Usage|
|---|---|---|
|General-purpose v2|Standard|Standard storage account type for blobs, file shares, queues, and tables. Recommended for most scenarios using Blob Storage or one of the other Azure Storage services.|
|Block blob|Premium|Premium storage account type for block blobs and append blobs. Recommended for scenarios with high transaction rates or that use smaller objects or require consistently low storage latency|
|Page blobs|Premium|Premium storage account type for page blobs only.|

# Access tiers for block blob data
Azure Storage provides different options for accessing block blob data based on usage patterns. Each access tier in Azure Storage is optimized for a particular pattern of data usage. By selecting the right access tier for your needs, you can store your block blob data in the most cost-effective manner.

The available access tiers are:

- The **Hot** access tier, which is optimized for frequent access of objects in the storage account. The Hot tier has the highest storage costs, but the lowest access costs. New storage accounts are created in the hot tier by default.
    
- The **Cool** access tier, which is optimized for storing large amounts of data that is infrequently accessed and stored for at least 30 days. The Cool tier has lower storage costs and higher access costs compared to the Hot tier.
    
- The **Archive** tier, which is available only for individual block blobs. The archive tier is optimized for data that can tolerate several hours of retrieval latency and will remain in the Archive tier for at least 180 days. The archive tier is the most cost-effective option for storing data, but accessing that data is more expensive than accessing data in the hot or cool tier
# Azure Blob Storage resource types
Blob storage offers three types of resources:

- The **storage account**.
- A **container** in the storage account
- A **blob** in a container

## Storage account
A storage account provides a unique namespace in Azure for your data. Every object that you store in Azure Storage has an address that includes your unique account name. The combination of the account name and the Azure Storage blob endpoint forms the base address for the objects in your storage account.

For example, if your storage account is named _mystorageaccount_, then the default endpoint for Blob storage is: `http://mystorageaccount.blob.core.windows.net`

## Container
A container organizes a set of blobs, similar to a directory in a file system. A storage account can include an unlimited number of containers, and a container can store an unlimited number of blobs.

A container name must be a valid DNS name, as it forms part of the unique URI (Uniform resource identifier) used to address the container or its blobs. Follow these rules when naming a container:

- Container names can be between 3 and 63 characters long.
- Container names must start with a letter or number, and can contain only lowercase letters, numbers, and the dash (-) character.
- Two or more consecutive dash characters aren't permitted in container names.
An example of blob container URI: `https://myaccount.blob.core.windows.net/mycontainer`

## Blobs
Azure Storage supports three types of blobs:

- **Block blobs** store text and binary data. Block blobs are made up of blocks of data that can be managed individually. Block blobs can store up to about 190.7 TiB.
- **Append blobs** are made up of blocks like block blobs, but are optimized for append operations. Append blobs are ideal for scenarios such as logging data from virtual machines.
- **Page blobs** store random access files up to 8 TB in size. Page blobs store virtual hard drive (VHD) files and serve as disks for Azure virtual machines.

An example of blob's URI is: `https://myaccount.blob.core.windows.net/mycontainer/myblob`

# Azure Storage security features
Azure Storage provides a comprehensive set of security capabilities that together enable developers to build secure applications:

- All data (including metadata) written to Azure Storage is automatically encrypted using Storage Service Encryption (SSE).
- Microsoft Entra ID and Role-Based Access Control (RBAC) are supported for Azure Storage for both resource management operations and data operations, as follows:
    - You can assign RBAC roles scoped to the storage account to security principals and use Microsoft Entra ID to authorize resource management operations such as key management.
    - Microsoft Entra integration is supported for blob and queue data operations. You can assign RBAC roles scoped to a subscription, resource group, storage account, or an individual container or queue to a security principal or a managed identity for Azure resources.
- Data can be secured in transit between an application and Azure by using Client-Side Encryption, HTTPS, or SMB 3.0.
- OS and data disks used by Azure virtual machines can be encrypted using Azure Disk Encryption.
- Delegated access to the data objects in Azure Storage can be granted using a shared access signature.
## Azure Storage encryption for data at rest

Azure Storage automatically encrypts your data when persisting it to the cloud. Encryption protects your data and helps you meet your organizational security and compliance commitments. Data in Azure Storage is encrypted and decrypted transparently using 256-bit AES encryption, one of the strongest block ciphers available, and is FIPS 140-2 compliant. Azure Storage encryption is similar to BitLocker encryption on Windows.

Azure Storage encryption is enabled for all new and existing storage accounts and can't be disabled. Because your data is secured by default, you don't need to modify your code or applications to take advantage of Azure Storage encryption.

Storage accounts are encrypted regardless of their performance tier (standard or premium) or deployment model (Azure Resource Manager or classic). All Azure Storage redundancy options support encryption, and all copies of a storage account are encrypted. All Azure Storage resources are encrypted, including blobs, disks, files, queues, and tables. All object metadata is also encrypted.

Encryption doesn't affect Azure Storage performance. There's no extra cost for Azure Storage encryption.

### Encryption key management

You can rely on Microsoft-managed keys for the encryption of your storage account, or you can manage encryption with your own keys. If you choose to manage encryption with your own keys, you have two options:

- You can specify a _customer-managed_ key to use for encrypting and decrypting all data in the storage account. A customer-managed key is used to encrypt all data in all services in your storage account.
- You can specify a _customer-provided_ key on Blob storage operations. A client making a read or write request against Blob storage can include an encryption key on the request for granular control over how blob data is encrypted and decrypted.
# Static web hosting using Azure Storage
You can serve static content (HTML, CSS, JavaScript, and image files) directly from a storage container named `$web`. Hosting your content in Azure Storage enables you to use serverless architectures that include Azure Functions and other Platform as a service (PaaS) services. Azure Storage static website hosting is a great option in cases where you don't require a web server to render content.

Static websites have some limitations. For example, If you want to configure headers, you have to use Azure Content Delivery Network (Azure CDN). There's no way to configure headers as part of the static website feature itself. Also, AuthN and AuthZ aren't supported. If these features are important for your scenario, consider using [Azure Static Web Apps](https://azure.microsoft.com/services/app-service/static/).
## Enable static web hosting
Static website hosting is a feature that you have to enable on the storage account. When you configure your account for static website hosting, Azure Storage automatically creates a container named `$web`. The `$web` container contains the files for your static website.

To enable static website hosting:

1. Locate your storage account in the Azure portal and display the account overview
2. Select **Static website** to display the configuration page
3. Select **Enabled** to enable static website hosting for the account
4. In the **Index document name** field, specify a default index page. For example, _index.html_.
5. In the **Error document path** field, specify a default error page. For example, _404.html_.
6. Select **Save**.

![static web host on azure storage](https://learn.microsoft.com/en-us/training/wwl-azure/explore-azure-blob-storage/media/enable-static-website-hosting.png)

You can modify the public access level of the _$web_ container, but making this modification has no impact on the primary static website endpoint because these files are served through anonymous access requests. That means public (read-only) access to all files.

While the primary static website endpoint isn't affected, a change to the public access level does impact the primary blob service endpoint.

For example, if you change the public access level of the _$web_ container from Private (no anonymous access) to Blob (anonymous read access for blobs only), then the level of public access to the primary static website endpoint `https://contosoblobaccount.z22.web.core.windows.net/index.html` doesn't change.

However, the public access to the primary blob service endpoint `https://contosoblobaccount.blob.core.windows.net/$web/index.html` does change from private to public. Now users can open that file by using either of these two endpoints.

Disabling public access on a storage account by using the public access setting of the storage account doesn't affect static websites that are hosted in that storage account. For more information, see Remediate anonymous public read access to blob data (Azure Resource Manager deployments).

## Mapping custom domains to static website URL
You can make your static website available via a custom domain.

It's easier to enable HTTP access for your custom domain, because Azure Storage natively supports it. To enable HTTPS, you have to use Azure CDN because Azure Storage doesn't yet natively support HTTPS with custom domains. Visit [Map a custom domain to an Azure Blob Storage endpoint](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-custom-domain-name) for step-by-step guidance.

# Authorize options for Azure Storage
## Public access
Public access is also known as anonymous public read access for containers and blobs.

There are two separate settings that affect public access:

- **The Storage Account.** Configure the storage account to allow public access by setting the _AllowBlobPublicAccess_ property. When set to true, Blob data is available for public access only if the container's public access setting is also set.
    
- **The Container.** You can enable anonymous access only if anonymous access has been allowed for the storage account. A container has two possible settings for public access: _Public read access for blobs_, or _public read access for a container and its blobs_. Anonymous access is controlled at the container level, not for individual blobs. So, if you want to secure some of the files, you need to put them in a separate container that doesn't permit public read access.
    
>Both storage account and container settings are required to enable anonymous public access. The advantages of this approach are that you don't need to share keys with clients who need access to your files. You also don't need to manage a SAS.

## MS Entra ID
Use the Microsoft Entra option to securely access Azure Storage without storing any credentials in your code. AD authorization takes a two-step approach. First, you authenticate a security principal that returns an OAuth 2.0 token if successful. This token is then passed to Azure Storage to enable authorization to the requested resource.

> Use this form of authentication if you're running an app with managed identities or using security principals.

## Shared keys
Azure Storage creates two 512-bit access keys for every storage account that's created. You share these keys to grant clients access to the storage account. These keys grant anyone with access the equivalent of root access to your storage.

> We recommend that you manage storage keys with Azure Key Vault because it's easy to rotate keys on a regular schedule to keep your storage account secure.

## Shared access signature
A SAS lets you grant granular access to files in Azure Storage, such as read-only or read-write access, expiration time, after which the SAS no longer enables the client to access the chosen resources. A shared access signature is a key that grants permission to a storage resource, and should be protected in the same manner as an account key.

Azure Storage supports three types of shared access signatures:

- **User delegation SAS**: Can only be used for Blob storage and is secured with Microsoft Entra credentials.
- **Service SAS**: A service SAS is secured using a storage account key. A service SAS delegates access to a resource in any one of four Azure Storage services: Blob, Queue, Table, or File.
- **Account SAS**: An account SAS is secured with a storage account key. An account SAS has the same controls as a service SAS, but can also control access to service-level operations, such as Get Service Stats.

You can create a SAS ad-hoc by specifying all the options you need to control, including start time, expiration time, and permissions.

If you plan to create a service SAS, there's also an option to associate it with a stored access policy. A stored access policy can be associated with up to five active SASs. You can control access and expiration at the stored access policy level. This approach is good if you need to have granular control to change the expiration, or to revoke a SAS. The only way to revoke or change an ad-hoc SAS is to change the storage account keys.