# DNS Client Configuration Options for Private Endpoints

## Introduction

The main goal of this article is to explore few specific scenarios and alternative DNS configuration options on the client side. If you are implementing Private Link/Endpoint you can leverage options listed here as alternative to make DNS client behavior changes based on your specific needs. the intention here is to illustrate two scenarios but you can leverage them for others too depending on you situatiom. Most of the exemples are related to specifically Private Endpoint (Consumers side) where private IP is allocated and name resolution plays a critical role to make it to work properly.

**Note:** It is important primarily to follow guidelines on [Private Endpoint DNS Integration Scenarios](https://github.com/dmauser/PrivateLink/tree/master/DNS-Integration-Scenarios). The intention of this document is to illustrate other customized options for specific scenarios and should be not use as primary guidance.

## DNS Client conditional forwarders

There are few options that can be configured in Operational System side to integrate them with Private Link/Endpoint. Below we will go over how customers can leverage those options as alternative or complement recommendations.

Operational Systems such as Windows and Linux have options to setup conditional forwarders directly on the client side.
On Windows this feature is called NRPT (Network Resolution Table) - More information see: [The NRPT](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn593632(v=ws.11)). On advantage on Windows is NRPT can also be set using Group Policy which makes easier to deploy for a group of computers that need to receive a specific setting.

On Linux this can be leverage using dnsmasq by adding rules conditional forward rules by editing /etc/dnsmasq.conf file. As side note dnsmasq is also refered as an recommended option to add caching capabilities to Linux OS, for more information see: [Getting the most from name resolution that Azure provides](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/azure-dns#getting-the-most-from-name-resolution-that-azure-provides)

## Scenario 1: On-Prem - Client direct resolve to Azure Custom DNS

In case you need only few computer in your network On-Prem network to be able to resolve Private Link private IP, you can have a local conditional forwarder on each computer to send a conditional fowarder resolve Private Link records in Azure Private DNZ via Azure Custom DNS Servers (in our exemple, using IPs 10.0.0.4 and 10.0.0.5).Azure Custom DNS Server configured to use 168.63.129.16 as Forwarder.
![nprtexample][nrptexample]
In this scenario, customer does not want to use their On-Prem DNS Server to resolve Private Link. This option is also a good alternative in case customer needs to make short term validation before adding conditional forwarder on their DNS Server.

### NRPT on Windows

In this PowerShell example a rule is added to redirect all traffic towards domain .core.windows.net to Custom DNS Servers 10.0.0.4 (Primary Azure Custom DNS Server) and 10.0.0.5 (Secondary Azure Custom DNS Server).

```Powershell
#Adding NRPT Conditional Forwarder rule
Add-DnsClientNrptRule -Namespace ".core.windows.net" -NameServers 10.0.0.4, 10.0.0.5
#Validation output
Get-DnsClientNrptRule | Format-Table
#Name resoltion validation
Resolve-DnsName "Storage-Account-FQDN" -DnsOnly -Type A | Format-Table -AutoSize
```

#### Output Examples

*Get-DnsClientNrptRule | Format-Table*
![Get-DnsClientNrptRule][image1]

*Resolve-DnsName*
![Resolve-DNSName][image2]

#### Group Policy

Same configuration above can be deployed via Group Policy (GPO), here is an exemple on how to make changes above over Local GPO (gpedit.msc):
![GroupPolicy][image3]

### Dnsmasq on Linux (Ubuntu)

This example shows same previous scenario but now for Linux  Unbuntu leveraging dnsmasq:

```Bash
#Install dnsmasq - Reference: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/azure-dns
sudo apt-get install dnsmasq

#(Optional) Rename default dnsmasq.conf file to dnsmasq.conf.bkp
sudo mv -v /etc/dnsmasq.conf /etc/dnsmasq.conf.bkp

# Addinng core.windows.net conditional forwarder rules
# Note 1: this can be done with any domain level or full dns name 
# Example: (FQDN) such as: blob.core.windows.net or windows.net or full storageaccount fqdn, i.e. stgspk1.blob.core.windows.net)
# Note 2: multiple lines or use editor commands such as nano or vim using example above: server=/FQDN-or-domain/FW-DNSServer
echo server=/core.windows.net/10.0.0.4 | sudo tee -a /etc/dnsmasq.conf
echo server=/core.windows.net/10.0.0.5 | sudo tee -a /etc/dnsmasq.conf

# Validate dnsmaq.conf content
cat /etc/dnsmasq.conf

# Make file dnsmasq.conf changes effective
sudo systemctl restart dnsmasq

# Update Resolver config file
sudo apt install resolvconf
sudo resolvconf -u

#Validate query (replace stgspk1 with your storage account name)
dig +noall +answer stgspk1.blob.core.windows.net

#(Optional - Validation) Make sure added conditional forwarder rules are effective on dnsmasq
sudo systemctl status dnsmasq
#(Optional - Troubleshooting) You may need flush local DNS Cache:
sudo systemd-resolve --flush-caches
```

### Command output examples

1) *dig +noall +answer stgspk1.blob.core.windows.net*
![dig][image4]
It shows query from OnPremLXVM1 going directly to Azure Custom DNS Servers.
2) *sudo systemctl status dnsmasq*
![dnsmasq][image5]

## Scenario 2: Conditional Forwarder exceptions

There are specific scenarios customer may need to create exceptions when dealing with Private Endpoints name resoltion when leveraging Azure Private DNS zones. One of these them is reported in this article: [Unable to access each other PaaS Resources when both sides are exposed to PrivateLink/Endpoint](https://github.com/dmauser/PrivateLink/tree/master/Issue-Customer-Unable-to-Access-PaaS-AfterPrivateLink). On that article one of the solutions, when name resolution does not work properly, is to have short term solution by creating a Conditional Forwarder to make sure Storage Account from Contoso gets resolved by its Public IP instead; for more details see: [Solution 1 - Create a conditional forwarder for storage account DNS name (FQDN)](https://github.com/dmauser/PrivateLink/tree/master/Issue-Customer-Unable-to-Access-PaaS-AfterPrivateLink#solution-1---create-a-conditional-forwarder-for-storage-account-dns-name-fqdn).
In case Custom DNS Server is not in place in your Azure you can also use same techniques explain on previous section by leveraging NRPT on Windows or dnsmasq on Linux to create same type of exceptions. Here is an exemple for both platforms using the same example of the article (Conditional Forwarder to OpenDNS Public Resolvers: 208.67.222.222 and 208.67.220.220) to get storage account name **contosostg1.blob.core.windows.net** resolved by its associated Public IP 52.230.240.94.

### Adding an exception on NRPT (Windows)

```Powershell
#Adding NRPT Conditional Forwarder rule
Add-DnsClientNrptRule -Namespace "contosostg1.blob.core.windows.net" -NameServers 208.67.222.222, 208.67.220.220
#Validation output
Get-DnsClientNrptRule | Format-Table
#Name resoltion validation
Resolve-DnsName "Storage-Account-FQDN" -DnsOnly -Type A  | Format-Table -AutoSize
```

#### NRP exceptions output Examples

*Get-DnsClientNrptRule | Format-Table*
![dnsmasq][image6]
*Resolve-DnsName*
![Resolve-DnsName][image7]

Same configuration above can be deployed via Group Policy (GPO), here is an exemple on how to make changes above over Local GPO (gpedit.msc):
![GPO][image8]

**Note:** make sure when creating over GPO interface make sure you select the correct DNS name type. In the example above because we use a full name **contosostg1.blob.core.windows.net** it should be FQDN type. On previous section it is done for domain **core.windows.net** and Suffix is specified as type.  

### Adding exceptions using dnsmasq (Ubuntu)

This example shows same previous scenario using NPTR but now for Linux  Unbuntu leveraging dnsmasq:

```Bash
#Install dnsmasq - Reference: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/azure-dns
sudo apt-get install dnsmasq

#(Optional) Rename default dnsmasq.conf file to dnsmasq.conf.bkp
sudo mv -v /etc/dnsmasq.conf /etc/dnsmasq.conf.bkp

# Addinng fqdn contosostg1.blob.core.windows.net as conditional forwarder
echo server=/contosostg1.blob.core.windows.net/208.67.222.222 | sudo tee -a /etc/dnsmasq.conf
echo server=/contosostg1.blob.core.windows.net/208.67.220.220 | sudo tee -a /etc/dnsmasq.conf

# Validate dnsmaq.conf content
cat /etc/dnsmasq.conf

# Make file dnsmasq.conf changes effective
sudo systemctl restart dnsmasq

# Update Resolver config file
sudo apt install resolvconf
sudo resolvconf -u

#Validate query (replace stgspk1 with your storage account name)
dig +noall +answer contosostg1.blob.core.windows.net

#(Optional - Validation) Make sure added conditional forwarder rules are effective on dnsmasq
sudo systemctl status dnsmasq

#(Optional - Troubleshooting) You may need flush local DNS Cache:
sudo systemd-resolve --flush-caches
```

### Command output examples after adding exceptions rules

1) *dig +noall +answer stgspk1.blob.core.windows.net*
![dig][image9]
It shows query from OnPremLXVM1 going directly to Azure Custom DNS Servers.
2) *sudo systemctl status dnsmasq*
![dnsmasq][image10]

### Other Possibilities

The intention of this article was to illustrate few DNS cleint configuration options when dealing with DNS resoltion when dealing with Private Endpoints. There are other potential possiiblities not enumerated here yet. Therefore, stay tuned for other potential scenarios.  

[image1]: ./media/image1.png
[image2]: ./media/image2.png
[image3]: ./media/image3.png
[image4]: ./media/image4.png
[image5]: ./media/image5.png
[image6]: ./media/image6.png
[image7]: ./media/image7.png
[image8]: ./media/image8.png
[image9]: ./media/image9.png
[image10]: ./media/image10.png
[nrptexample]: ./media/nrptexample.png