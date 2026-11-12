# 01 — Final Network Topology

## Evidence

- Image: `01_Final_Network_Topology.png`
- Project file: `07_Packet_Tracer/Three-Site-Enterprise-Network-Lab.pkt`
- Platform: Cisco Packet Tracer

## What it shows

A headquarters, two branch offices and a simulated ISP. HQ holds the server and the central services;
each branch has one router and one access switch; all three enterprise sites connect over serial WAN
links, and HQ reaches the ISP over an Ethernet link.

## Caption

I designed and built this simulated three-site enterprise network to connect a headquarters and two
branch offices through a routed WAN. I placed the server and network services at HQ, used dedicated
switches and VLANs at each site instead of one flat network, and gave every VLAN its gateway through
router-on-a-stick subinterfaces on the site router. Both branches exchange routes with HQ over OSPF
and reach the internet through NAT at the HQ edge.

## Sites

### Headquarters

- Routers: R1-HQ
- Switches: SW1-HQ-CORE, SW2-HQ-USERS, SW3-HQ-SERVICES
- Endpoints: six Sales PCs, three IT PCs, HQ-MANAGEMENT-PC
- Server and printer: HQ-SERVER01, HQ-PRINTER01

### Branch 1

- Router: R2-BR1
- Switch: SW4-BR1
- Endpoints: three PCs
- Printer: BR1-PRINTER01

### Branch 2

- Router: R3-BR2
- Switch: SW5-BR2
- Endpoints: two PCs

### ISP and external network

- Router: R0-ISP
- Server: INTERNET-SERVER01

## Links

- R1-HQ G0/0 to SW1-HQ-CORE G0/1 — trunk carrying the HQ VLANs
- SW1-HQ-CORE Fa0/23 to SW2-HQ-USERS Fa0/23 — trunk
- SW1-HQ-CORE Fa0/24 to SW3-HQ-SERVICES Fa0/23 — trunk
- R1-HQ S0/0/0 to R2-BR1 S0/0/0 — serial WAN, R1 is the DCE end
- R1-HQ S0/0/1 to R3-BR2 S0/0/0 — serial WAN
- R1-HQ G0/1 to R0-ISP G0/0 — routed link to the ISP
- R0-ISP G0/1 to INTERNET-SERVER01 — external network
- R2-BR1 G0/0 to SW4-BR1 G0/1 — trunk
- R3-BR2 G0/0 to SW5-BR2 G0/1 — trunk

## Counts

- Routers: 4
- Switches: 5
- PCs: 15
- Printers: 2
- Servers: 2
- Total: 28

The four user networks use 100-address DHCP scopes from .100 to .199 in their own /24.
