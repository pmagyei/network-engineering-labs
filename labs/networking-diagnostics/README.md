This document is a record of my mental models before starting the networking labs

## Scenario A — Ethernet, ARP and same-subnet forwarding

/24 means the 1st 24 bits identify the network, the remaining 8 identify the host

Host B 192.168.10.50/24 is in the same subnet


an arp message identifies the mac address of the ip 
When host A pings .50 for the first time:

an arp message is sent to the switch, essentially host A is asking "who does 192.168.10.50 belong to?"

Host A arps for the switch's mac address which is the destination of the ARP request

The switch then broadcasts the arp message throughout the network for a reply

Host B receives the request and replies to the switch with its mac address

the switch learns HOST B mac address and populates the its ARP table. 

When the ICMP packet is sent the destination is changed to HOST b MAC address

The default gateway participates by forwarding the frames and broadcasting the arp message to learn mac address assigments



## Scenario B — VLANs, trunks and inter-VLAN routing


pc1 - sw1 - L3 switch - sw2 - pc2

PC1 arps for PC2, the destination MAC is the interface of Vlan20 on L3 switch

The IP destination is PC2

The arp message is sent to sw1, sw1 forwards and tags the frmae on the trunk SW1 > L3 switch

L3 switch removes the tag and inspects the frame, the routing table is inpspected on  L3 switch to determine where the frame should go, routes and tags the frame accordingly, in this case to pc2 on VLAN 20

pc2 sends a reply, sw2 populates its arp table, tags the frame a nd forwards to L3 Switch

L3 switch removes the tag and inspects the frame, the routing table is inpspected, routes and tags the frame to sw1, sw1 removes the tag and forwards the frame to pc1, pc1 learns the mac - ip assignment of PC2, now it sends a icmp packet to the gateway, the gateway checks the routing table, routes the packet to interface vlan 20, the packet is sent to pc2

## Scenario C — Default gateway failure


fault 1
no default route, 
observation: no default route on linux vm
command: `ip route`

fault 2
missing route table entry, 
observation: no static route on router,
command: `show ip route`

fault 3
power failure

## Scenario D — Routing decision

10.20.30.10     192.168.1.3  longest prefix match
10.20.30.150    192.168.1.3  longest prefix match
10.20.99.50     192.168.1.2  longest prefix match
10.99.1.20      192.168.1.1  longest prefix match
172.16.1.10     192.168.1.254 gateway of last resort

When multiple route entries have the same destination AD and Metric are used to dertermine the next hope


## Scenario E — ACL versus stateful firewall

The statefull firewall can track source traffic and return taffic

a statement to allow https from 10.10.10.0/24 to 172.16.10.10.

and another statemtn to allow https from 172.16.10.10. to 10.10.10.0/24

a basic acl cannot track the port used

## Scenario F — DHCP


Discover

Clients send a broadcast message to DHCP servers asking for a lease

Offer

DHCP server replies to client with an ip(offer)

Requests

Client requests the IP to be used, DHCP makes sure that the leased IP is not already in use

Ackwoledge

Client confirms IP 

DHCP relay/IP helper:

This allows to forward UDP/TCP accross the network boundaries


## Scenario G — DNS


I understand DNS conceptually but not enough to answer how to troubleshoot it


## Scenario H — NAT/PAT


PAT

allows multiple IP addresses to use one IP address on the internet. it works by assigning unique ports on conection. the firewall identifies the internal hosts by the ports assigned on each connection.

NAT only translates the addresses over the internet, packet routing is an entirely separate function.


## Scenario I — EtherChannel/LACP

Give me a diagnostic sequence.

For each stage tell me what command/output you would inspect and what evidence would support or weaken the hypothesis.


check the interface configuration of both switch ports `show int g0/* brief`, inspect the interface state if its down, ensure that interface characteristics are identical such as vlans/native vlan, trunk/access, speed/duplex,


vlans/native vlan
vlan mismatches would prevent tagged frames from moving across the link, a native vlan only allows untagged frames, both swicth ports configuration need to be identical or the port-channel will not form

trunk/access

a trunk port allows tagged/untagged frames to traverse the link

an access port is usually conencted directly to the client

if one side of the link is an access port and the other is a trunk port, the port-channel will not form due to port mode inconsistency 

I would inspect vlan inconsistencies using `show interfaces trunk`


determine that the lacp negotiation on both switch ports is the same, `show etherchannel summary`



physical member - this are the individual switch ports 
logical port-channel - this is formed once the individual swtiched ports are bundled together in a group
LACP negotiation - both switch ports sends messages hello/dead timers to each other to establish port aggregation

## Scenario J — SNMP

SNMP polling:

pulls for messages from the clients

SNMP traps/informs:

pushes messages to the server 


The monitoring server could have failed to pull the messages from the switch



## Final baseline question


What changes were made before Finance VLAN could not access the file server?

Check where the file server reside(IP/subnet and VLAN), establish its identity

Is the FINANCE VLAN traffic tagged correctly, is a trunk port configured? L2

Check route table, s the file server still reacheable from finance VLAN IP pool? L3

acls, Is finance VLAN allowed to reach it? 


## Incident reconstruction — Part 1: chronology

1. OBSERVATION - I attempted to access cisco CML through the browser
2. OBSERVATION - I inspected tailscale and the router(opnsense) was offline, the green dot dissapered, this meant I had no remote access to my home infrstructure
3. HYPOTHESIS - I suspected that the network went down becasue opnsense went offline
4. OBSERVATION - when i reached the server rack, everything was powered down apart from the ups and the switch
5. TEST - I manually turned on the server, then proceded to turn off the mains, the server turned off
6. VERIFICATION/ACTION - docs and google search suggested that I had plugged the devcices into the surge protected outlets instead of the battery backed outlets. I unplugged the power, made sure everything was turned off and i plugged the devices into the battery backed outlets
7. OBSERVATION - The switch and the server powered on, I used a terminal cable to confirm the switch's status, I inspected the etherchannel betwen the switch and the NAS, and it was still down.
8. HYPOTHESIS - The UPS had abruptly turned off, it failed to power on automatically after a power failure, this cause the lacp negotiation to fail
9. ACTION - I manually turned on the NAS, waited a few minutes, inspected the switch, confirmed that the etherchannel came back online
10. OBSERVATION - I was unable to start an VM's on proxmox even though the NAS was powered on and functioning
11. HYPOTHESIS -  Proxmox could not reach the NAS due network misconfiguration
12. TEST - I pinged the NAS from proxmox it did not work
13. ACTION - inspect opnsense, dhcp assigned a new IP address
14. EVIDENCE - the ip lease assignment tab showed a new ip address for the NAS, this explained why vm's could not start
15. ACTION - updated the NAS IP on proxmox
16. VERIFICATION - restarted proxmox, and attempted to start a VM, it was able to boot
17. ACTION - to prevent dynamic IP. re-assignment to my core infrstructure I impleemented static assignment


## Part 2: dependency model

Then independently reconstruct this dependency chain:

Remote client
    ↓
Tailscale
    ↓
OpnSense
    ↓
Cisco CML


and separately:
Proxmox
    ↓
proxmox receives network connectivity from opnsense which runs as vm, if the vm goes down so does the network.
    ↓
Synology NFS service

tailscale runs within opnsense and advertises the networks I need to access


## Part 3: EtherChannel evidence

I check the incident documentation, I checked the etherchannel summary on the switch before powering on the NAS, hence the etherchannel was still down, no lacp re-confirguration was made. 


## Part 4: NFS failure
I want a packet-level explanation of this section.
Suppose Proxmox expected the NAS at:
192.168.x.10
but DHCP caused the NAS to return after reboot as:
192.168.x.25


Explain why this can break NFS without calling NFS a Layer-3 protocol.
Tell me:
- what dependency actually failed
- whether the server IP is part of the application's endpoint/configuration
- what Layer-3 functionality may still have been perfectly healthy
- why changing DHCP behaviour can prevent recurrence

Prxmox expexts a NFS device at 192.168.x.10, having the NAS returning as something diffrent, prevents proxmox from reaching the NAS

the virtual machines depend on the NAS for storage through network connectivity, if proxmox is unable to reach the NAS the VM's cannot start

storage and network dependency failed, network went temporarily down, NAS powered off abrutply. Even though IP assignment was healthy, dynamic ip assignment caused an impartial recovery of my inrstructure.




updating the new IP address of the NAS on proxmox was the immidite remediation to restore services, however if the ip lease expired this would cause IP re-assignment


Implementing static IP assigments to prevent dynamic ip leasing ensured that each core device persistently maintains an IP address persistently


I have also created a diagnostics directory within the github repository 
for the diagnostic tests, the mental models that are weak sould be strenthened by creating labs, documenttion and commit to the repository, i noticed you did not include ipv6 in your diagnostic questions, include ipv6 moving forward wherer necessary and benicial