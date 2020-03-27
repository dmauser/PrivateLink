## Known Issue
Azure Customers are unable to access each other PaaS Resources when both sides are exposed to PrivateLink/Endpoint.

## Consider a scenario:
Contoso Corporation is using Private Endpoint to access their Storage Accounts, example blog storage account name: **contosostg1**.
 - ContosoVM1 -> Resolves Storage Account using privatelink.blob.core.windows.net to get private IP associated to Private Endpoint in order to access contosostg1.blob.core.windows.net

In the other side Fabrikam is also using Private Endpoints to access their own Storage Accounts, for example fabriakamstg1, using same exact scenario as Contoso.
 - FabrikamVM1 -> Resolves Storage Account using privatelink.blob.core.windows.net to get private IP associated to Private Endpoint in order to access fabrikamstg1.blob.core.windows.net

For this scenario both customers host their respective privatelink.blob.core.windows.net using Azure Private DNS Zones.

Contoso wants allow Fabrikam to access their container named orders under storage account contosostg1.blob.core.windows.net

Contoso expects Fabrikam to access their storage account using Public IP. However, when Fabrikam IT team tries to resolve contosostg1.blob.core.windows.net from FabrikamVM1 they detect that resolution does not work at all as shown:

![](./Media/image1.png)

1. Unable to ping contosostg1.blob.core.windows.net (name resolution failed to obtain IP)
2. Nslookup doesn't show error but no IP is resolved.
3. Resolved-DNSName cmdlet shows resolution stops when CNAME contosostg1.privatelink.blob.core.windows.net is triggered

## Root Cause:
FabrikamVM1 is unable to resolve that contosostg1.blob.core.windows.net because is also using Private Link and has also CNAME for its name fabrikamstg1.privatelink.blob.core.windows.net. Because Fabrikam also hosts that Azure Private DNS zone privatelink.blob.core.windows.net it will be unable to get is Public IP.

Note that If you execute the same tests outside of Fabrikam environment (not integrated with Private DNS Zone for privatelink.blob.core.windows.net) resolution will bring Public IP fine.
    
You can validate that by either running Nslookup, Resolve-DnsName (PowerShell) or Dig against contosostg1.blob.core.windows.net as shown:

![](./Media/image2.png)

1. Public IP is returned fine.
2. You can confirm with this extra CNAME contosostg1.privatelink.blob.core.windows.net that storage account is exposed to Private Link/Endpoint (For more information on how DNS Name resolution works with PrivateLink here: [How DNS resolution works before and after Private Endpoints](https://github.com/dmauser/PrivateLink/tree/master/DNS-Integration-Scenarios#2-how-dns-resolution-works-before-and-after-private-endpoints)

## Recommended Solution: 

Expose **contosostg1** storage account via Private Link/Endpoint to Fabrikam.
Fabrikam has to create a Private Endpoint and request Private Link to contosostg1.blob.core.windows.net. After that DNS record for contosostg1 inside AzurePrivate DNS zone privatelink.blob.core.windows.net in order to access.

 **Note**: We may have other solutions that will be added later in this document.

In order to create PrivateEndpoint on Fabrikam side Contoso has to provide them Storage Account contosostg1 ResourceID. In my example:
/subscriptions/SubscriptionID/resourceGroups/RGTEST/providers/Microsoft.Storage/storageAccounts/contosostg1

### 1. Using that information Fabrikam start the process to create a Private Endpoint on their side using Azure Portal ###
(Screenshots in sequence - Basics, Resource, Configuration, Review+Create):

- **Step 1** - Add Private Private Endpoint

![](./Media/image3.png)

- **Step 2** - Select Subscription, Resource Group, Private Endpoint Name and Region.

![](./Media/image4.png)

- **Step 3** - On resource tab you have to add Storage Account Resource ID provided by Constoso. Example: /subscriptions/(Add **SubID**)/resourceGroups/RGTEST/providers/Microsoft.Storage/storageAccounts/contosostg1

![](./Media/image5.png)

- **Step 4** - Add target Virtual Network where Private Endpoint will reside. **Note** At this point Private DNS is not configured. You will to add record manually after you get Private Link request approved by Contoso.

![](./Media/image6.png)

- **Step 5** - Review all details and process by Create.

![](./Media/image7.png)

### 2. After Private Endpoint get Created you will see:

 1. Private Link Resource showing Storage Account name contosostg1.
 2. Connection Status as Pending because Contoso still need to approve Private Endpoint Request from Fabrikam
 3. DNS full name will be shown only after Contoso approval

![](./Media/ContosoPep-Pending.png)

### 3. Contoso now has to approve that request that came from Fabrikam. Steps below done over Azure Portal on Contoso side.

![](./Media/image8.png)