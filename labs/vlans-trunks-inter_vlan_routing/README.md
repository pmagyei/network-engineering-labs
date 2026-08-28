# VLANs, trunks and inter-VLAN routing

Requirement:

Host A VLAN 10
Host B VLAN 20

SW1 must carry both vlan frames to R1 over a single Ethernet Link

R1 must provide L3 path routing between the two networks.



## Host A pings 10.10.20.20

Packet Path Reasoning:

Host A decides:
10.10.20.20 is remote to host A's IP 10.10.10.10
Host A needs 10.10.20.20 to resolve the ARP
The ethernet destination is FF:FF:FF:FF::FF:FF

Host A > SW1:
The frame traverses VLAN 10, at staged no frames are being tagged with 802.1Q
SW1 learns Host A's Mac address

SW1 > R1:
As the frame leaves the trunk it is tagged to distinguish it from other frames coming from other VLANs, the will have VLAN 10 tagged on
the trunk.
The ethernet destination is FF:FF:FF:FF::FF:FF

R1
the frame's tag is removed 
the router must decide where the next hop is
the ethernet frame is decapsulated to reveal the ip packet payload

R1 > SW1 > Host B
R1's ethernet port
Host B's mac becomes the new ethernet destination
VLAN 20
it is present when it travel the trunk port and it is removed when it exits

Return Path

Host B sends the reply through the default gateway because:
there's a vlan boundary, VLAN 10 cannot directly communicate to VLAN 20

network boundary, the packets must be routed to reach other networks.