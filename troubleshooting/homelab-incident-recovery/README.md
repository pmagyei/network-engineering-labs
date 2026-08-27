
## Home lab Physical/ Network Topology

Physical/network topology

## dependency diagram

When remotely accessing:

Remote client
    ↓
Tailscale
    ↓
OpnSense
    ↓
Cisco CML


Infrastructure dependency:

Proxmox
    ↓
proxmox receives network connectivity from opnsense which runs as vm, if the vm goes down so does the network.
    ↓
Synology NFS service

tailscale runs within opnsense and advertises the networks I need to access

## Incident chronology

verified, inferred, and unknown/not proven.

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


## Remote-access dependency

## Proxmox/OPNsense relationship

## NAS/storage dependency


storage and network dependency failed, network went temporarily down, NAS powered off abrutply. Even though IP assignment was healthy, dynamic ip assignment caused an impartial recovery of my inrstructure.

## EtherChannel observations

## DHCP address-change evidence

## NFS recovery

## UPS outlet investigation

## Recovery verification


updating the new IP address of the NAS on proxmox was the immidite remediation to restore services, however if the ip lease expired this would cause IP re-assignment

## Preventive controls

Implementing static DHCP assigments/reservation to prevent dynamic ip leasing, ensuring that each core device persistently maintains an IP address persistently