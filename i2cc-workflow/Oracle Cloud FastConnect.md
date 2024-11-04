# Oracle Cloud FastConnect

::: tip
Both Oracle Government cloud (OC2) and Oracle commercial cloud (OC1) FastConnect connections are completed by selecting Oracle as the cloud provider. The cloud type is encoded in the FastConnect OCID, so Virtual Networks will use the site where the FastConnect OCID is valid.
:::

A note on peering addresses:

- Oracle IPv4 BGP peering addresses are in a block with subnet mask ranging from `/28` to `/31`. See the instructions in [FastConnect: With an Oracle Partner](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/fastconnectprovider.htm#FastConnect_With_an_Oracle_Partner)
- Oracle IPv6 BGP peering addresses must be in a block with a subnet mask of `/64`, `/96`, `/126` or `/127`. See [FastConnect and IPv6](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/ipv6.htm#fastconnect)

Workflow:

In the Oracle Cloud console:

- Under Networking>FastConnect, create a new FastConnect (FastConnect Partner) using either `Internet2: Internet2 L2` or `Internet2: Internet2 L3` as the provider, depending on if you want a Layer 2 (connection to a Virtual Switch) or Layer 3 (connection to a Virtual Router) virtual network within Internet2.
  - For Layer 2 you will be asked to input all BGP parameters in the Oracle console
    - IPv4 settings are required, IPv6 settings are optional
    - The BGP key will be the same for IPv4 and IPv6
  - For Layer 3 all BGP parameters are specified within Virtual Networks
  - Choose Private virtual circuit
  - Select the desired bandwidth and MTU
    - bandwidth is also specified in the Virtual Networks console; they should match, but if they differ the Virtual Networks specification wins.
    - if you later edit the bandwidth in the Oracle Console, Virtual Networks should pick up the change within an hour
- Copy the OCID of the FastConnect and wait for the virtual circuit (VC) to go to PENDING PARTNER from PROVISIONING.

Similar to Azure, for Layer 3 connections you can re-use the FastConnect OCID once and create a secondary connection. If you want a resilient Layer 2 connection you need to create a second FastConnect. The BFD setting is common between the primary and secondary; the BGP key could be different between primary and secondary (but is common between any IPv4 and IPv6 peering on the same connection).

In the Virtual Networks section of Internet2 Insight Console:

- Create a new connection to A cloud provider, choose Oracle, and provide the FastConnect OCID.
- Select the `Continue` Arrow
- Select the region the FastConnect was created in
- Choose the interface and VLAN you would like to use (typically this is the VLAN toward your infrastructure, but there is no need for it to be the same, the Internet2 Network will translate), and select the `Continue` arrow again
- for Virtual Router (Layer 3) connections, Specify the IPv4 and optionally IPv6 peering addresses, subject to the rules above. The MTU and max bandwidth of the connection are specified as well; note that the bandwidth specified here should match the bandwidth specified in the Oracle console, and if it is different it overrides that in the Oracle console (although you can also update the bandwidth of an existing connection in the Oracle console). You can also specify a BGP key.

## Deprovisioning

Deleting a connection in Virtual Networks will result in the Oracle FastConnect VC terminating, ending in it being completely deleted as well.