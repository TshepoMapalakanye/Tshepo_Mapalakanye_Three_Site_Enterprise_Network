# Three-Site Enterprise Network Design and Implementation

**Cisco Packet Tracer | Enterprise Networking Lab**
Tshepo Mapalakanye

## Project overview

I designed and configured a simulated three-site enterprise network in Cisco Packet Tracer,
connecting a headquarters and two branch offices over a routed WAN with a simulated ISP and an
external web server. I wanted a network I could explain end to end rather than a configuration I had
copied: segmentation first, then routing, then centralised services, then verification of every part.

## Architecture and technologies

Four routers, five switches, fifteen workstations, two printers and two servers. Sales, IT, servers,
printers, branch users and management each sit in their own VLAN and subnet, with VLAN 999 as an
unused native VLAN on every trunk. Each site router provides its VLAN gateways through
router-on-a-stick subinterfaces, and serial /30 links join the branches to HQ.

OSPF area 0 carries the routes between the three enterprise routers, with passive interfaces by
default and a default route originated from HQ towards the ISP. HQ-SERVER01 provides DHCP across
four scopes of 100 addresses with relay from every client subnet, plus DNS, HTTP and FTP. NAT
overload sits at the HQ edge, SSH version 2 provides device administration, and the access switches
run spanning tree with a set root bridge and port security with sticky learning.

## Verification

All sixteen planned tests passed. They covered gateway reachability at each site, DHCP allocation
from all four scopes, DNS resolution, internal and external web access, printer reachability, OSPF
adjacencies and the learned default route at each branch, NAT translations, SSH to all eight
enterprise devices from the management PC, and port-security learning. I recorded the command output
and the result of each test rather than treating a configured feature as a working one.

## Troubleshooting

Seven faults came up during the build. The first was a native VLAN mismatch on the SW1 to SW2 trunk:
the core switch used native VLAN 999 while the downstream switch was still on VLAN 1. Tagged traffic
kept crossing, so no endpoint reported a problem, and only CDP showed it. I compared both ends with
`show interfaces trunk`, corrected the downstream switch, applied the same values to the second HQ
trunk, and confirmed the warning cleared. Each fault is written up in the log by layer, cause, fix and
lesson.

## Skills developed

Structured switching and routing design, OSPF and NAT troubleshooting, centralised service
configuration with DHCP relay, secure device administration, and the habit of proving a fix instead
of assuming it.
