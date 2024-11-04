# Microsoft Azure ExpressRoute

::: tip
Both Azure Government and Azure commercial ExpressRoute connections are completed by selecting Microsoft Azure as the cloud provider. Virtual Networks will query both provider sites and proceed using the site where the Service Key is valid.
:::

An Azure ExpressRoute has two connections to diverse routers by default. ExpressRoute calls these "Primary" and "Secondary", although they are treated like equal-cost multipath peers. The first time through the Virtual Neworks workflow, the primary connection is created. You re-use the Service Key and go through the workflow a second time to create the secondary connection. (Creating the secondary connection is not required, but is encouraged for resiliency.)

We do not recommend connecting the primary and secondary connections to the same virtual switch. For virtual routers, standard practice is to connect the primary and secondary to the same virtual router (although using separate virtual routers is also possible).

Workflow:

In the Azure Portal:

- Create an ExpressRoute in the Azure portal. Choose Internet2 as the provider, and the Peering Location where the ExpressRoute is created.

  | Azure Peering Location | Internet2 PoP       |
  | ---------------------- | ------------------- |
  | Washington DC          | Ashburn VA (ASHB)   |
  | Chicago                | Chicago IL (EQCH)   |
  | Dallas                 | Dallas TX (DALL3)   |
  | Silicon Valley         | San Jose, CA (SANJ) |

  - The size of the ExpressRoute connection is only specified inside the Azure portal. It can be raised, but never lowered in the future. (One can always create a new ExpressRoute with a lower bandwidth and delete the old ExpressRoute.)

- Copy the "Service Key" once the ExpressRoute circuit has been created

In Virtual Networks (repeat for the secondary connection):

- Use the Service Key in Virtual Networks after selecting cloud provider and Microsoft Azure to create a connection to an ExpressRoute interface
- Azure dictates which interface is used
- For Virtual Switch connections, select the `Continue` arrow, and then you can optionally enter a description, then cick `Save Draft`
- For Virtual Router connections, you can enter an inner VLAN ID (this is arbitrary, so it can be anything from 1 to 4095; often schools match a VLAN number facing the school), select the `Continue` arrow, and then you are brought to a pane to configure the BGP peering information. Enter the information and then select `Save Draft` to continue. **Virtual Networks will only configure private peering for ExpressRoute Virtual Router connections.**
- Azure is contacted as soon as the draft is saved to find the peering location, interface used, and bandwidth configured.
- You must select `Go Live` for the connection to be configured on the Internet2 Network.

**Note:** When configuring IPv4 peer addresses, Azure expects a /30 for both the primary and secondary connections. The first address will be used by the peer and the second will be used by Azure. For example, if 192.2.0.248/30 is used, 192.2.0.249/30 will be used by the peer and 192.2.0.250/30 will be used by Azure.

Similarly, when configuring IPv6 peer addresses, Azure expects a /126 for both the primary and secondary connections (not a /64). The first usable address is used for your equipment (or Internet2) and the second useable address is used by Azure. For example, if 2001:db8:468::0/126 is used, 2001:db8:468::1/126 will be used by the peer and 2001:db8:468::2/126 will be used by Azure. For more information, as of this writing see [IP Addresses Used for Azure Peering](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-routing#ip-addresses-used-for-peering).

::: warning
As of this writing Azure does not allow configuring a BGP Shared Key, so that is not available in Virtual Networks.
:::

## Deprovisioning

In order for Virtual Networks to be able to deprovision an ExpressRoute connection, all attachments must be removed from the ExpressRoute connection in the Azure console. This includes any virtual gateways, Global Reach connections, and even authorizations on the ExpressRoute circuit. Only then will Virtual Networks be allowed to deprovision the ExpressRoute connection completely.

Once that is done, you are able to delete the ExpressRoute in the Azure console and stop Azure billing.

Unlike other providers, Azure allows a deprovisioned ExpressRoute to be used again if it is not deleted. To reuse the ExpressRoute, start the workflow over from the top, once again using the Service Key for the ExpressRoute circuit.