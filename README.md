# Three-Site Enterprise Network — Cisco Packet Tracer

A simulated enterprise network connecting a headquarters and two branch offices over a routed WAN,
with a simulated ISP and an external web server. I built it from an empty Packet Tracer workspace,
tested it end to end and documented the faults I hit along the way.

## What this is

I wanted a network I could explain rather than a configuration I copied. That meant designing the
addressing and segmentation first, building the switching and routing on top of it, putting the
services where they belong, and then proving each part worked with a command or a test rather than
assuming it did.

- Headquarters: R1-HQ, three switches, six Sales PCs, three IT PCs, a management PC, the central
  server and a printer.
- Branch 1: R2-BR1, one switch, three PCs and a printer.
- Branch 2: R3-BR2, one switch and two PCs.
- ISP: R0-ISP and INTERNET-SERVER01.

HQ holds the services and the internet edge. Both branches reach HQ over serial links and learn
their routes with OSPF.

## Design in short

Sales, IT, servers, printers, branch users and management each have their own VLAN and subnet.
VLAN 999 is used as the native VLAN on every trunk and carries no hosts, so untagged frames cannot
land in a user VLAN. Every VLAN gateway is a router-on-a-stick subinterface on the site router. The
WAN links are /30 point-to-point serial links, and R1-HQ originates a default route into OSPF so both
branches know how to reach the simulated internet. NAT overload sits on R1's ISP-facing interface,
with an access list that only selects which internal sources are translated.

## What I configured

- VLANs 10, 20, 30, 40, 50, 60, 99 and native 999, with access ports and explicit allowed-VLAN lists
  on every trunk.
- Router-on-a-stick inter-VLAN routing at all three sites.
- OSPF area 0 between the three enterprise routers, with passive interfaces by default and only the
  WAN links enabled.
- Static default route on R1-HQ toward the ISP, redistributed into OSPF.
- DHCP on HQ-SERVER01 with four scopes of 100 addresses and `ip helper-address` relay on each
  client-facing subinterface, plus DNS, HTTP and FTP on the same host.
- NAT overload (PAT) at the HQ edge.
- SSH version 2, local administrator accounts and `transport input ssh` on the VTY lines.
- Spanning Tree with SW1-HQ-CORE as the root bridge, and port security with sticky learning on the
  access switches.

## Verification

All sixteen planned tests passed. They covered each site's gateway, DHCP allocation from all four
scopes, DNS resolution, internal and external web access, printer reachability, OSPF adjacencies and
the learned default route at each branch, NAT translations on R1-HQ, SSH to all eight enterprise
devices from HQ-MANAGEMENT-PC, and port-security learning on the access switches.

A configured feature is not a verified feature, so the evidence in this repository is command output
and test results rather than configuration listings alone.

## Troubleshooting

Seven faults came up during the build, and `05_Troubleshooting/Troubleshooting_Log.md` indexes
them with a full account of each. The first was a native VLAN mismatch on the SW1 to SW2 trunk: the
core switch was set to native VLAN 999 while the downstream switch was still on VLAN 1. Tagged
traffic kept crossing, so nothing at the endpoints noticed, and CDP reported it on the console. I
compared both ends with `show interfaces trunk`, corrected the downstream switch, applied the same
values to the SW1 to SW3 trunk, and confirmed the warning cleared.

Each write-up works up the OSI model from the physical layer, then records the fix and what the fault
taught me.

## Repository layout

- `01_Topology/` — final topology and how the sites connect.
- `02_Architecture_and_Addressing/` — device roles, VLANs, subnets and addressing plan.
- `03_Router_Configurations/` — interface, OSPF, routing and NAT evidence.
- `04_Switch_Configurations/` — VLAN, trunk, STP and port-security evidence.
- `05_Troubleshooting/` — the incident log and one write-up per fault.
- `06_Project_Report/` — the one-page report and its PDF.
- `07_Packet_Tracer/` — the Packet Tracer project file.

— Tshepo Mapalakanye
