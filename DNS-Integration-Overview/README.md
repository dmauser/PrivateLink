# Private Endpoint DNS Integration Overview (Draft)

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


