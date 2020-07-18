# Private Link DNS Scenario using Active Directory

## Content
Introduction](#introduction)
[Scenario](#scenario) 
[Deploying the solution](#deploying-the-solution)
[Azure side](#azure-side)
[LAB](#lab)
[Closing](#closing)

## Introduction

Several companies use Active Directory (AD) as their primary authentication service. One of the core components that to make AD run properly is Domain Name Services (DNS) and brings a lot of several advantages such as a full feature DNS as well as replication of DNS zones using AD replication. Another capability of DNS in AD environments is the capability to make DNS zone application partitions which allows as well as define replication scopes.

Specifically for Private Link/Endpoint integration you need to create conditional forwarders zones in On-Premises to reach

If you don't have a fully understanding on how Private Link/Endpoint works, it is recommended you review the following article: [Private Endpoint DNS Integration Scenarios](https://github.com/dmauser/PrivateLink/tree/master/DNS-Integration-Scenarios).

## Scenario

In this scenario with Active Directory customer wants to integrate Azure Private DNS which requires customer to create a conditional forwarder zone to blob.core.windows.net integrated with AD. The biggest advantage of using this approach is customer does not require to create a Conditional Forwarder individually in each Domain Controller running DNS Server. Instead, customer creates a single Conditional Forwarder Zone and integrate it with Active Directory application partition. With that integration the Conditional Forwarder zone will be replicated automatically to all Domain Controllers.

## Deploying the solution

Below is the commands used to make this solution to work on the scenario above:

### Azure side

1. Define variables Powershell variables
 
```Powershell
$AZDNSADPartition="AzureDNS" #Name of the AD Application Partition
$AZDC1="HUBWDC1" #DC1 in Azure
$AZDC2="HUBWDC2" #DC2 in Azure
$CFZ="blob.core.windows.net" 
```

2. Creating DNS Application Partition
```
Add-DnsServerDirectoryPartition -Name $AZDNSADPartition
```

3. Create Conditional Fowarder Zone integrated to AD 

In this case a conditional forwarder to: blob.core.windows.net will be created and 168.63.129.16

```Powershell
Add-DnsServerConditionalForwarderZone -ComputerName $AZDC1 -Name $CFZ -ReplicationScope Custom -DirectoryPartitionName $AZDNSADPartition -MasterServers 168.63.129.16
```

4. Register
```
Register-DnsServerDirectoryPartition -Name $AZDNSADPartition -ComputerName $AZDC2
```

5. Validate
```
Get-DnsServerDirectoryPartition -Name $AZDNSADPartition -ComputerName $AZDC1
Get-DnsServerDirectoryPartition -Name $AZDNSADPartition -ComputerName $AZDC2
```

Expected output:

## LAB

A Lab for this scenario will be included soon, stay tunned.

## Closing

Thanks for Azure Networking GBB Team for reviewing this content and special thanks for Microsoft FTE Clive Graven.
