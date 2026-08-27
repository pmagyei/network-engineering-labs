# Ethernet, MAC forwarding and ARP

Event1


Ethernet source MAC: AA-AA-AA-AA-AA-AA
Ethernet destination MAC: BB-BB-BB-BB-BB-BB
ARP sender IP: 10.10.10.10/24
ARP sender MAC: AA-AA-AA-AA-AA-AA
ARP target IP: 10.10.10.20/24
ARP target MAC: BB-BB-BB-BB-BB-BB


SW1 learns host A mac, and populates cam table, assigning switchport(Gi0/1) to host mac


Event 2

SW1 consults the CAM table, it broadcasts the frame to all the active switchports, when it receives a reply from the target host it also populates the cam table the target host's mac

the switch does not modify the ethernet source/destination however it does forward the frames to the intended source and make sure the reply is sent back to the source

event 3

Ethernet source: BB-BB-BB-BB-BB-BB
Ethernet destination: AA-AA-AA-AA-AA-AA
ARP sender IP: 10.10.10.20/24
ARP sender MAC: BB-BB-BB-BB-BB-BB
ARP target IP: 10.10.10.10/24
ARP target MAC: AA-AA-AA-AA-AA-AA

Host A learns Host B's mac, and populated the arp table, assigning the MAC to the IP address
SW1 learns Host B's mac address came from Gi0/2

Event 4

Source IP: 10.10.10.10/24
Destination IP: 10.10.10.20/24
Source MAC: AA-AA-AA-AA-AA-AA
Destination MAC: BB-BB-BB-BB-BB-BB

Different concepts are identified in the same packet/frame because of the ARP table being populated.