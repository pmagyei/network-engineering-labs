# Ethernet, MAC forwarding and ARP

Host A > SW1 > Host B

Ethernet source MAC: 52:54:00:60:ed:b6
Ethernet destination MAC: FF:FF:FF:FF:FF:FF
ARP sender IP: 10.10.10.10
ARP sender MAC: 52:54:00:60:ed:b6
ARP target IP: 10.10.10.20
ARP target MAC: 00:00:00:00:00:00


SW1 learns Host A mac on (Gi0/1); A switch learns the source MAC address of frames arriving on a port.

SW1 sees Ethernet destination MAC: `FF:FF:FF:FF:FF:FF`, it floods the frame because that is the ethernet broadcast address.

The switch does not modify the ethernet source/destination MAC address.

Host B replies because the ARP payload asked for the device owning `10.10.10.20`


Host B > SW1 > Host A

Ethernet source: 52:54:00:b8:ef:67
Ethernet destination: 52:54:00:60:ed:b6
ARP sender IP: 10.10.10.20
ARP sender MAC: 52:54:00:b8:ef:67
ARP target IP: 10.10.10.10
ARP target MAC: 52:54:00:60:ed:b6


The return path:
Host B learns Host A mac address from initial ARP request
source mac address is BB, destination mac address is: AA
SW1 learns: `52:54:00:b8:ef:67 → Gi0/1`
SW1 already know that: `52:54:00:60:ed:b6 → Gi0/0`

SW1 CAM/MAC table:

`5254.0060.edb6 → Gi0/0`
`5254.00b8.ef67 → Gi0/1`


Host A's ARP table: `10.10.10.20 → 52:54:00:b8:ef:67`



`Host A sends an IP packet to 10.10.10.20`

Source IP: 10.10.10.10
Destination IP: 10.10.10.20
Source MAC: 52:54:00:60:ed:b6
Destination MAC: 52:54:00:b8:ef:67

When Host A has a IP packet to send to 10.10.10.20/24

/24 tells Host A that 10.10.10.20 is in the same subnet so it does not send the IP packet to the default gateway, instead it needs an Ethernet destination MAC for link-local delivery.

ARP provides the mapping:

10.10.10.20 → BB-BB-BB-BB-BB-BB

SW1 checks the CAM/MAC table: `52:54:00:b8:ef:67 → Gi0/2`, forwards the frame to `Gi0/2`

ARP does not need to re-occur again while Host A retains a valid ARP cache entry for 10.10.10.20. Similarly this applies to the switch CAM/MAC entries.

Host A encapsulates the IP packet within an Ethernet frame destined for Host B.


[CML-LAB](./evidence/Ethernet-MAC-ARP.png)