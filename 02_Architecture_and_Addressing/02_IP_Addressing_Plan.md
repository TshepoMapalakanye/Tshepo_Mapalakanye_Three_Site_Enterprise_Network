# 02 — IP Addressing Plan

Every subnet in the lab, its gateway and how its addresses are allocated. Internal addressing uses
private ranges; the WAN handoff and external network use documentation ranges.

## HQ networks

### VLAN 10 — Sales

- Network ID: 192.168.10.0/24
- Usable addresses: 254
- Gateway: 192.168.10.1 on R1-HQ G0/0.10
- DHCP scope: 192.168.10.100 to 192.168.10.199, 100 addresses

### VLAN 20 — IT

- Network ID: 192.168.20.0/24
- Usable addresses: 254
- Gateway: 192.168.20.1 on R1-HQ G0/0.20
- DHCP scope: 192.168.20.100 to 192.168.20.199, 100 addresses

### VLAN 30 — Servers

- Network ID: 192.168.30.0/24
- Usable addresses: 254
- Gateway: 192.168.30.1 on R1-HQ G0/0.30
- Addressing: static only
- HQ-SERVER01: 192.168.30.10

### VLAN 60 — HQ Printers

- Network ID: 192.168.60.0/24
- Usable addresses: 254
- Gateway: 192.168.60.1 on R1-HQ G0/0.60
- Addressing: static only
- HQ-PRINTER01: 192.168.60.10

### VLAN 99 — HQ Management

- Network ID: 192.168.99.0/24
- Gateway: 192.168.99.1 on R1-HQ G0/0.99
- Addressing: static only

## Branch 1 networks

### VLAN 40 — BR1 Users

- Network ID: 192.168.40.0/24
- Usable addresses: 254
- Gateway: 192.168.40.1 on R2-BR1
- DHCP scope: 192.168.40.100 to 192.168.40.199, 100 addresses

### VLAN 60 — BR1 Printers

- Network ID: 192.168.61.0/24
- Gateway: 192.168.61.1 on R2-BR1
- Addressing: static only
- BR1-PRINTER01: 192.168.61.10

### VLAN 99 — BR1 Management

- Network ID: 192.168.91.0/24
- Gateway: 192.168.91.1 on R2-BR1
- Addressing: static only

## Branch 2 networks

### VLAN 50 — BR2 Users

- Network ID: 192.168.50.0/24
- Usable addresses: 254
- Gateway: 192.168.50.1 on R3-BR2
- DHCP scope: 192.168.50.100 to 192.168.50.199, 100 addresses

### VLAN 99 — BR2 Management

- Network ID: 192.168.92.0/24
- Gateway: 192.168.92.1 on R3-BR2
- Addressing: static only

## WAN links

### R1-HQ to R2-BR1

- Network ID: 192.168.200.0/30
- Addresses: 4 total, 2 usable
- R1-HQ: 192.168.200.1
- R2-BR1: 192.168.200.2
- Broadcast: 192.168.200.3

### R1-HQ to R3-BR2

- Network ID: 192.168.200.4/30
- R1-HQ: 192.168.200.5
- R3-BR2: 192.168.200.6
- Broadcast: 192.168.200.7

### R1-HQ to R0-ISP

- Network ID: 203.0.113.0/30
- R1-HQ: 203.0.113.1
- R0-ISP: 203.0.113.2
- Broadcast: 203.0.113.3
- R1 default route points at 203.0.113.2

The /30 links carry exactly two usable addresses, one per router.

## External network

### External LAN

- Network ID: 198.51.100.0/24
- Gateway: 198.51.100.1 on R0-ISP
- INTERNET-SERVER01: 198.51.100.10

## Static infrastructure addresses

### Management PC

- HQ-MANAGEMENT-PC: 192.168.99.10/24, gateway 192.168.99.1, DNS 192.168.30.10

### Switch management addresses

- SW1-HQ-CORE: 192.168.99.11
- SW2-HQ-USERS: 192.168.99.12
- SW3-HQ-SERVICES: 192.168.99.13
- SW4-BR1: 192.168.91.11
- SW5-BR2: 192.168.92.11

### Server and printers

- HQ-SERVER01: 192.168.30.10/24, gateway 192.168.30.1, DNS 192.168.30.10
- HQ-PRINTER01: 192.168.60.10, gateway 192.168.60.1
- BR1-PRINTER01: 192.168.61.10, gateway 192.168.61.1

### External server

- INTERNET-SERVER01: 198.51.100.10, gateway 198.51.100.1

## Evidence

- `DHCP_Scopes.png` — the four user scopes on HQ-SERVER01, with the Sales scope open showing its .100
  start address, gateway and DNS, so the ranges above can be read straight off the server
