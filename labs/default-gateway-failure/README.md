# Default Gateway Failure


## Topology 

This lab consist of 2 Ubuntu host, 2 Switches and 1 router

[](./topology/topology.png)

All hosts have been configured with ip addresses

## Prediction


When Host A pings 10.10.20.20 what happens id default gateway is missing:

 the initial arp request will fail, as there is no default gateway to send the L2 frame.

1. Host A determines that 10.10.20.20 by looking at the network portion of the address; both subnets are using /24. 

2. it needs the default gateway/route, route table answers: where is this going next?

3. No default route, will prevent arp request, as a result the host cannot forward packets because route look up fails

4. No frame is sent SW1

When Host A pings 10.10.20.20 what happens if default gateway is incorrect:

5. ARP request would be sent only throughout Host A's local Layer-2 broadcast domain/VLAN, the default gateway will not reply to the ARP request if the IP is different to the one configured.


### Behavioural differences: Missing vs Incorrect

6. 

#### Incorrect default gateway
If Host A cannot resolve gateway Mac:

To get to `10.10.20.20` the next hop is > `10.10.10.254`
- A route is present to make L3 forwarding. however when Host A arp's for `10.10.20.254` it will not learn the mac address of the default gateway if nobody responds, L2 frame construction fails. ICMP packet does not get forwarded, but ARP frames do leave Host A and reach SW1.

Observable evidence:

- no default gateway: packets are dropped, no packets are traversing the network from Host A


#### Missing default gateway
If Host A has no remote destination:
- With no route for remote destination, selection for next hop cannot occur, subsequently no ARP is sent.

7.

Observable evidence:
- incorrect default gateway: ARP request sent but no reply




