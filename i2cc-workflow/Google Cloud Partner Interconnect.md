# Google Cloud Partner Interconnect

Internet2 PoPs are not bound to particular GCP regions, unlike AWS or Oracle. Any GCP region is accessable from any Virtual Networks interconnect site (the point of presence, or PoP). Generally you create the VLAN attachment in the same region as your GCP Cloud Router. Choose the closest Internet2 PoP that is on the way to that GCP region from the connection to the Internet2 network, or another if you want additional resiliency from redundant attachments.

| Internet2 PoP      | Google facility name | Low-latency region |
| ------------------ | -------------------- | ------------------ |
| ASHB - Ashburn VA  | iad-zone{1,2}-1      | us-east4           |
| EQCH - Chicago IL  | ord-zone{1,2}-7      |                    |
| DALL3 - Dallas TX  | dfw-zone{1,2}-4      | us-south1          |
| SANJ - San Jose CA | sjc-zone{1,2}-6      |                    |

See [Google region and zone documentation](https://cloud.google.com/compute/docs/regions-zones) and [Google colocation facility documentation](https://cloud.google.com/network-connectivity/docs/interconnect/concepts/choosing-colocation-facilities) for more information.

Each Internet2 PoP has interconnects to two GCP zones. When you create a VLAN attachment in the UI, GCP chooses the zone, which is encoded in the pairing key. If you create a pair of VLAN attachments (which GCP recommends when you need the highest resiliency) it will create two pairing keys, each in a different zone. If the VLAN attachments are requested using the GCP API (or CLI), you can specify the zone or say that either zone is fine (and the key ends in "/any".

Workflow:

In the GCP Console:

- Create a VLAN attachment (or a pair of VLAN attachments) in the Google Cloud console.
  - Note that the MTU should match that of your network attached to the cloud router (1440, 1460, 1500 or 8896 bytes)
  - Copy the pairing key(s).
    - note that the ending of the key specifies the zone; /1 specifies zone 1, and /2 specifies zone 2. In the API, but not the console, you can also say "any zone", in which case /any specifies any zone.

In the Virtual Networks console, do the following for each pairing key. If you created two VLAN attachments, we do not recommend connecting both to the same virtual switch. For virtual routers, standard practice is to connect both VLAN attachments to the same virtual router (which in turn will be connected two two different physical devices, one for each zone). You may also connect each VLAN attachment to a separate virtual router.

- Use the pairing key in Virtual Networks after selecting cloud provier and Google Cloud Platform to create a connection to the associated VLAN attachment.
- Select an interface that has enough remaining bandwidth free. **We ask that you select a 10G interface if you are requesting less than 1G of bandwidth, otherwise select a 100G interface**
  - bandwidth is specified in Virtual Networks in the next step
  - interfaces are offered that are in the Zone your key requests, but are unordered with respect to location. Typically you select an interface close to the region on the path from your connection to the Internet2 network to that region (although again the interfaces are not bound to region, so you can for example request an Ashburn VA interface to get to the us-west2 region).
  - Select the VLAN you would like to use.
  - Select the `Continue` arrow.
  - For Virtual Switch connections you specify the bandwidth
  - For Virtual Router connections you can specify bandwdith and whether or not you would like to use BFD, and the BGP key. The rest of the parameters are specified by GCP, and are read by Virtual Networks.
    - You can update whether BFD is used and the BGP key later.
    - In GCP, the BGP key is specified in the peering, so it may be easier to do in a subsequent step after you verify the peering comes up without a key.
      ::: tip
      GCP allows different keys for IPv4 and IPv6 peerings, but Virtual Networks currently only allows a single key for both. Therefore if you allow GCP to pick a key for one peering, you need to copy that key to the other peering (if it exists) as well as copying it in to Virtual Networks.
      :::
  - Select `Save Draft`; at this time GCP is contacted to find out the rest of the parameters associated with the service key, and to instantiate the VLAN attachment.
  - When provisioning is finished, you will either see `Connection provisioned, please activate in your GCP account` or `[Provisioned]` if the connection was pre-activated (see below).
  - You must select `Go Live` for the connection to be configured on the Internet2 Network.

Back in the GCP Console:

- Go back to the Google Cloud console and activate the VLAN attachment (acknowledging you are willing to pay for the specified bandwidth). You can also "pre-activate", and avoid this step.

## Deprovisioning

If you delete the connection in Virtual Networks, the state goes to `[GCP] [PENDING] - Tearing down connection`, and is removed after a few minutes. In the Google console, the VLAN attachment becomes `defunct`. You then need to delete the VLAN attachment.