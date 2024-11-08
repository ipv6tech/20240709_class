# Amazon Web Services

AWS calls a hosted cloud connect connection a "hosted connection".

Workflow:

- Create a new connection in Virtual Networks (Choose "A cloud provider", then "AWS"), specifying an AWS account number that will hold the hosted connection.
  - Select the region the hosted connection will directly connect to (us-east-1 (which we interconnect in Ashburn VA and Dallas TX), us-east-2 (we interconnect in Chicago) or us-west-1 (we interconnect in San Jose CA). Note that you can get to us-west-2 from any other region.
  - Next pick an interface with sufficient capacity. Choosing different devices for multiple connections provides additional redundancy. **We ask that you pick a 10G interface if you are allocating less than a 1G hosted connection, and 100G otherwise** (where 100G is available - Ashburn and San Jose currently)
  - Finally you are given the opportunity to select the bandwidth, and then BGP peering parameters for Virtual Router connections.
  - When the virtual network is saved, the AWS Console associated with the AWS account number is offered a new hosted connection.
  - The size of the AWS hosted connection is specified in the Virtual Networks console initially, and **cannot be changed**. You must create a new hosted connection to change the bandwidth.
- Accept the hosted connection in the AWS console
- Attach a AWS Virtual Interface (VIF) to the hosted connection. This will specify the IP addresses, local ASN (from a virtual gateway or direct connect gateway), and remote ASN (55038 for Internet2 connections with a virtual router), if BFD is going to be used, and if a BGP key is going to be used.

::: tip
Virtual Networks currently (Sept 2023) has the restriction that IPv4 and IPv6 BGP keys be identical. If you create both IPv4 and IPv6 peerings, if you let AWS choose the BGP key for the first peering, remember to copy it into the second peering you create for the other address family. If you let AWS choose both keys, they will be different and one will not come up.

To change a BGP key within AWS, you need to delete the old peering, wait for it to actually delete, then add a new peering for that address family. If you do this, the other address family (the one not changed) peering will remain up and passing traffic. The delete is traffic impacting for the family deleted.

If you delete and recreate an IPv6 peering, the addresses AWS chooses for the pering remain the same, even if the BGP key changes (at least it has in our tests). Be aware that it _could_ also change, though.
:::

- For connections to a virtual router, return to Virtual Networks and update the AWS peering with any information that was auto-generated by AWS (IP addresses, BGP key, ASN...) or updated
  - The MTU must match the AWS VIF (1500 for standard Ethernet, 9001 for virtual private gateways with jumbo frames or 8500 for transit gateways with Jumbo frames)
- You must select `Go Live` for the connection to be configured on the Internet2 Network.

## Deprovisioning

Before Virtual Networks can delete a hosted connection, the VIF must be deleted from the connection in the AWS Console. If this not done, an error will be shown in the Provisioner pane of the Details page for the connection.
