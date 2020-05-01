# Private Endpoint DNS Integration Overview (Draft)

## 1. What is Private Link/Endpoint?

The main goal os this article is to provide a brief overview of Private Link/Endpoint and how DNS plays an important role to role. However, before moving on DNS resolution for Private Link/Endpoint let's take a step back to explain what is Private Link and Endpoint. In short Private Link is a new Azure capability to acess a variaty Azure resources in Private way. By design most of Plataform as a Service (PaaS) offerings by Azure had in the past only accessibility via Public IP addresses which

If you want to get more familiarized with Private Link and Private Endpoint fuctionality it is highly recommend watch this Private Link session Ignite 2019 session: [Delivering services privately in your VNet with Azure Private Link](https://myignite.techcommunity.microsoft.com/sessions/BRK3168?source=TechCommunity).

## 2. Why DNS plays an important role on Private Link/Endpoint?

The main function of DNS name resolution is to translate names in IP addresses for target resources. When resolving DNS names for PaaS Services (example, Storage Account name or Azure SQL Database) by default you are going to get a Public IP.

Examples:
1) Azure Storage Account name gbbstg1 for blob will get the following result:
nslookup gbbstg1.blob.core.windows.net

2) Azure SQL Database named sql123 will get the following result:
nslookup sql123.database.windows.net

Therefore, the main goal when you expose a Azure resource (either PaaS or your own Service) via PrivateLink is to get the same exact target name but in this case resolved to private IP address. For the examples above, when they are exposed via Private Link and internal clients (either on Azure Private Networ or On-Premises) they will resolve PaaS name and get its private IP address.

Examples:
1) Internal Name resolution with Private Linkd

## 3. Azure Virtual Network DNS integration

Azure Private DNS is the best solution to integrate Private Link/Endpoint to be consumed properly. That allows customers to expand Private Endpoint get resolved properly in a global way. Azure Private DNS is global resource and allows Virtual Networks of any region to be linked to it (up to 1000 Virtual Networks can be linked). That will allow any Azure resources such as Azure Virtual Machines to proper resolve the private IP of exposed PaaS resousource over Private Link/Endpoint.

Customers who are levering Azure Provided DNS server (168.63.129.16) on their Virtual Network will be able to resolve all Private Endpoint resources without any extra configuration needed.

There are other scenarios such as using Custom DNS Server and how they are configured which can affect how Private DNS zone resolution can work properly. Those scenarios are covered on: [Private Endpoint DNS Integration Scenarios](https://github.com/dmauser/PrivateLink/tree/master/DNS-Integration-Scenarios).

## 4. On-Premises DNS integration



## 5. Putting all toghether Azure VNET and On-Premises 

