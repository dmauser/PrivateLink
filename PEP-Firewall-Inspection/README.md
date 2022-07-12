# Private Endpoint inspection patterns 
(coming soon)


## Background

## Notes:

Microsoft official reference for Azure Firewall:

## PE inspection patterns over Firewall (Azure Firewall or NVA)

|||||
| Client| NVA/Firewall location| Private Endpoint location | NAT required | Details |

On premises	On Premises	Hub or Spoke	No	 traffic will return to on premises using the same path
On premises	Hub	Hub or Spoke	No	 traffic from service will return to NVA/Firewall normally
Spoke1	Hub	Spoke2	No	traffic from service will return to NVA/Firewall normally as Spoke1 does not have any PE routes
Hub	Hub	Hub	Yes	traffic from service will return to client directly creating asymmetric behavior given that it has PE routes
Spoke1	Hub	Spoke1	Yes	traffic from service will return to client directly creating asymmetric behavior given that it has PE routes
Spoke1	Hub	Hub	Yes	traffic from service will return to client directly creating asymmetric behavior given that it has PE routes

