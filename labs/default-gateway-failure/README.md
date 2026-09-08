# Default Gateway Failure


## Topology 

This lab consist of 2 Ubuntu host, 2 Switches and 1 router

[](./topology/topology.png)

All hosts have been configured with ip addresses

## Prediction


When host A pings 10.10.20.20, the initial arp request will fail, as there is no default gateway to send the L2 frame.

1. Host A determines that 10.10.20.20 by looking at the network portion of the address; both subnets are using /24. 

2. it needs the default gateway/route, route table answers: where is this going next?

3. No default route, will prevent arp request, as a result packets will be dropped


4. No frame is sent SW1

5. ARP request is sent through out the network, however the default gateway will reply to the ARP request if the IP is different to the one configured.

6. 
- cant resolve mac, it fails to send L2 frames as they are instantly dropped
- no route for remote destination, it can send and ARP request, but the default gateway's IP is different so it will not send an ARP reply.

7.

- no default gateway: packets are dropped, no packets are traversing the network from Host A

- incorrect default gateway: ARP request sent but no reply




