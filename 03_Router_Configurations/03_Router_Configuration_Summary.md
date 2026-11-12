# 03 — Router Configuration Summary

What was configured on each router and which screenshot proves it.

## R1-HQ

### Role

HQ gateway for five VLANs, hub for both branch WAN links and the internet edge.

### VLAN gateways

- G0/0: trunk to SW1-HQ-CORE, enabled, no address
- G0/0.10: 192.168.10.1
- G0/0.20: 192.168.20.1
- G0/0.30: 192.168.30.1
- G0/0.60: 192.168.60.1
- G0/0.99: 192.168.99.1

The user subinterfaces also relay DHCP to 192.168.30.10 for their 100-address scopes.

Subinterfaces rather than SVIs because the 2960s are Layer 2 only, so the router already on the path
for the WAN and the internet is the natural place for every VLAN gateway.

### WAN links

- S0/0/0 to R2-BR1, 192.168.200.0/30, DCE end, clock rate 2000000
- S0/0/1 to R3-BR2, 192.168.200.4/30

### Internet edge

- G0/1 to R0-ISP, 203.0.113.1/30
- Default route 0.0.0.0/0 to 203.0.113.2, advertised into OSPF
- NAT overload: ACL 1 selects 192.168.0.0/16 sources, `ip nat inside` on the internal interfaces,
  `ip nat outside` on G0/1

The NAT ACL selects addresses for translation. It is not a traffic filter, and no ACL in this build
blocks traffic.

### Evidence

- `R1_Interface_Status.png`
- `R1_OSPF_Neighbors.png`
- `R1_OSPF_Protocols.png`
- `R1_Routing_Table.png`
- `R1_NAT_Statistics_&_Translations.png`

## R2-BR1

### Role

BR1 gateway and WAN endpoint.

### Configuration

- G0/0.40: 192.168.40.1, DHCP relay to 192.168.30.10
- G0/0.60: 192.168.61.1
- G0/0.99: 192.168.91.1
- S0/0/0: 192.168.200.0/30 to HQ

### Evidence

- `R2_Interface_Status.png`
- `R2_Routing_Table.png`

## R3-BR2

### Role

BR2 gateway and WAN endpoint.

### Configuration

- G0/0.50: 192.168.50.1, DHCP relay to 192.168.30.10
- G0/0.99: 192.168.92.1
- S0/0/0: 192.168.200.4/30 to HQ

### Evidence

- `R3_Interface_Status.png`
- `R3_Routing_Table.png`

## R0-ISP

### Role

Simulated provider edge and route to the external server.

### Configuration

- G0/0: 203.0.113.2/30 towards R1-HQ
- G0/1: 198.51.100.1/24 towards INTERNET-SERVER01

### Evidence

- `R0_Interface_Status.png`

## OSPF

- Area 0 on all three enterprise routers
- Unique router ID per router; process IDs are local only
- `passive-interface default`, with OSPF neighbours enabled on the two WAN links only
- Network statements using wildcards, for example `network 192.168.200.0 0.0.0.3 area 0`
- HQ originates the default route into OSPF, so both branches learn `O*E2 0.0.0.0/0`

Result: R1 shows both neighbours in FULL state, and both branch routing tables carry HQ's networks
plus the default route.

## Baseline on all four routers

Hostname, banner, enable secret, local administrator account, SSH with RSA keys and a domain name,
VTY lines accepting SSH only, management path, and the configuration saved to startup.
