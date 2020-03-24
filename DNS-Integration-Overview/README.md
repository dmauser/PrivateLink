## 1. What is Private Link/Endpoint?

The main goal os this article is to provide a brief overview of Private Link/Endpoint and how DNS plays an important role to role. However, before moving on DNS resolution for Private Link/Endpoint let's take a step back to explain what is Private Link and Endpoint. In short Private Link is a new Azure capability to acess a variaty Azure resources in Private way. By design most of Plataform as a Service (PaaS) offerings by Azure had in the past only accessibility via Public IP addresses which

If you want to get more familiarizaed with Private Link and Private Endpoint it highly recommend you what this Ignite session on how Private Link/Endpoint will allow to access Azure Services fully private

## 2. Why DNS plays an important role on Private Link/Endpoint?

The main function of DNS name resolution is to translate names in IP addresses for target resources. When resolving DNS names for PaaS Services (example, Storage Account name or Azure SQL Database) by default you are going to get a Public IP.
Example:
1) Azure Storage Account name gbbstg1 for blob will get the following result:
nslookup gbbstg1.blob.core.windows.net


2) Azure SQL Database named sql123 will get the following result:
nslookup sql



Therefore, the main goal when you expose a Azure resource (either PaaS or your own Service) is to get via name resolution resolving to the right private IP insteawill play an important role because 

## 3. DNS Integration Scenario for Azure VNET

Azure Private DNS is the best solution to integrate Private Link/Endpoint to be consumed properly. That allows customers to expand Private Endpoint get resolved properly in a global way. Azure Private DNS is global resource

## 4. DNS Integration Scenario for On-Premises

## 5. Putting all toghether Azure VNET and On-Premises 

Private Link and Private Endpoints has been a long waited feature by Azure customers to allow fully private connectivity to Azure PaaS Services. By design, most of Azure PaaS services are public endpoints and other options have been offered to facilitate customers to secure their access to them such as [Virtual Network (VNET) Service Endpoints](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview) and [VNET Integration](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-for-azure-services). However, Private Link/Endpoint offers a complete solution to access Azure PaaS Service fully private.

Before we start to go in more DNS resolution process for Private Link and Private Endpoints, it is important to clarify important terminologies used in this article. First, that Private Endpoint represents **Service Consumer** side as well as where name resolution (DNS) is associated with. In other hand, Private Link represents **Service Provider** side which connects to a Private Endpoint to be properly accessed. If you are not familiar with Private Link and Private Endpoints please review official documentation at [http://aka.ms/privatelink](http://aka.ms/privatelink). Additionally, in Service Provider side there are three types of serviced that can exposed using Private Link: Azure PaaS Services, Partner Services and Customer Owned Services. This article will cover DNS integration scenarios for Private Endpoint when connecting via Private Link with Azure PaaS Services only and more specifically Azure Blob Storage Accounts to illustrate most of the examples and same logic can be applied to other PaaS services.

DNS integration has also different aspects to be enumerated when dealing with resolution inside Azure Virtual Network (VNET) versus On-Premises, and which one presenting its own considerations and challenges depending on customer's DNS usage footprint. Therefore, this article will be separated by: How DNS resolution works before and after private endpoint, Azure Virtual Network integrations and On-Premises DNS integrations scenarios with Private Endpoint. It is important to mention this content may not cover all scenarios, but it will go over the simplest implementation up to more complex ones which can give you an idea on how properly make your scenario to work with Private Endpoints. The first take away from document is to understand how DNS resolution works as well as how to asses DNS configuration for customers will help you to establish the best solution.
