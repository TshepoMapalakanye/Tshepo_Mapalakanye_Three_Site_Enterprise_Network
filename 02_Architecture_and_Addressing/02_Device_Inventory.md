# 02 — Device Inventory

Every device in the lab and what it does.

## Headquarters

### R1-HQ

- Type: Cisco 2911 router
- Role: enterprise edge and HQ gateway
- Duties: router-on-a-stick gateways for five HQ VLANs, both branch WAN links, default route to the
  ISP and NAT overload

### SW1-HQ-CORE

- Type: Cisco 2960 switch
- Role: HQ core switch
- Duties: trunks to the router and both access switches, management VLAN, spanning-tree root

### SW2-HQ-USERS

- Type: Cisco 2960 access switch
- Role: HQ user access
- Ports: FastEthernet0/1 to 0/6 Sales in VLAN 10, FastEthernet0/7 to 0/9 IT in VLAN 20

### SW3-HQ-SERVICES

- Type: Cisco 2960 access switch
- Role: HQ service access
- Ports: FastEthernet0/1 server in VLAN 30, FastEthernet0/2 printer in VLAN 60

### HQ-MANAGEMENT-PC

- Type: management workstation
- Role: administration and verification
- Addressing: static in VLAN 99; used for SSH to every network device

### Sales PCs

- Type: six user endpoints
- Role: VLAN 10 traffic, DHCP from the Sales scope

### IT PCs

- Type: three user endpoints
- Role: VLAN 20 traffic, DHCP from the IT scope

### HQ-SERVER01

- Type: server
- Role: central services
- Duties: DHCP for four user scopes, DNS, HTTP, FTP
- Addressing: static in VLAN 30

### HQ-PRINTER01

- Type: printer
- Role: HQ print services, static address in the HQ printer subnet

## Branch 1

### R2-BR1

- Type: Cisco 2911 router
- Role: BR1 gateway and WAN endpoint
- Duties: subinterfaces for VLAN 40, VLAN 60 and VLAN 99, serial link to HQ

### SW4-BR1

- Type: Cisco 2960 access switch
- Role: BR1 access
- Ports: FastEthernet0/1 to 0/3 users in VLAN 40, FastEthernet0/4 printer in VLAN 60

### BR1 user PCs

- Type: three user endpoints
- Role: VLAN 40 traffic, DHCP from the BR1 scope

### BR1-PRINTER01

- Type: printer
- Role: branch print services, static address in the BR1 printer subnet

## Branch 2

### R3-BR2

- Type: Cisco 2911 router
- Role: BR2 gateway and WAN endpoint
- Duties: subinterfaces for VLAN 50 and VLAN 99, serial link to HQ

### SW5-BR2

- Type: Cisco 2960 access switch
- Role: BR2 access
- Ports: FastEthernet0/1 and 0/2 users in VLAN 50

### BR2 user PCs

- Type: two user endpoints
- Role: VLAN 50 traffic, DHCP from the BR2 scope

## ISP and external network

### R0-ISP

- Type: Cisco 2911 router
- Role: simulated service provider
- Duties: terminates the handoff from R1-HQ and serves the external network

### INTERNET-SERVER01

- Type: external web server
- Role: simulated public destination for routing and NAT tests

## Totals

- Routers: 4
- Switches: 5
- PCs: 15
- Printers: 2
- Servers: 2
- Total: 28
