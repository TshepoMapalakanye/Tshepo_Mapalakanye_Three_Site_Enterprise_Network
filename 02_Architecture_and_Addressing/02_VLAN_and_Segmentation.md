# 02 — VLANs and Network Segmentation

## VLANs in use

### VLAN 10 — Sales

- Purpose: HQ sales endpoints
- Subnet: 192.168.10.0/24

### VLAN 20 — IT

- Purpose: HQ IT endpoints
- Subnet: 192.168.20.0/24

### VLAN 30 — Servers

- Purpose: the HQ server network
- Subnet: 192.168.30.0/24
- Hosts: DHCP, DNS, HTTP and FTP on HQ-SERVER01

### VLAN 40 — BR1 Users

- Purpose: branch 1 endpoints
- Subnet: 192.168.40.0/24

### VLAN 50 — BR2 Users

- Purpose: branch 2 endpoints
- Subnet: 192.168.50.0/24

### VLAN 60 — Printers

- Purpose: printer traffic at HQ and BR1
- Subnets: 192.168.60.0/24 at HQ, 192.168.61.0/24 at BR1

### VLAN 99 — Management

- Purpose: device management addresses
- Subnets: one per site, 192.168.99.0/24, 192.168.91.0/24 and 192.168.92.0/24

### VLAN 999 — Native

- Purpose: untagged frames on the trunks
- Subnet: none

VLAN 1 carries no user traffic and is not the native VLAN anywhere.

## DHCP scopes

### Sales

- Scope: 192.168.10.100 to 192.168.10.199
- Addresses: 100

### IT

- Scope: 192.168.20.100 to 192.168.20.199
- Addresses: 100

### BR1 users

- Scope: 192.168.40.100 to 192.168.40.199
- Addresses: 100

### BR2 users

- Scope: 192.168.50.100 to 192.168.50.199
- Addresses: 100

Each pool is set by a start address of .100 and a maximum user count of 100, so it ends at .199.
Addresses .2 to .99 and .200 to .254 stay free for static use, which keeps the gateway, printers and
management addresses out of the dynamic range.

## Trunks

- Access ports carry one VLAN; trunk ports carry several using 802.1Q tags
- Every trunk lists its allowed VLANs explicitly
- Every trunk uses native VLAN 999, which carries no user traffic and has no IP address
- Trunks exist between SW1 and SW2, SW1 and SW3, SW1 and R1-HQ, SW4 and R2-BR1, SW5 and R3-BR2

## Router-on-a-stick

One physical interface per router carries a trunk, with a subinterface acting as the gateway for each
VLAN.

### R1-HQ

- GigabitEthernet0/0: trunk to SW1-HQ-CORE, no address of its own
- G0/0.10: 192.168.10.1
- G0/0.20: 192.168.20.1
- G0/0.30: 192.168.30.1
- G0/0.60: 192.168.60.1
- G0/0.99: 192.168.99.1

### R2-BR1

- G0/0.40: 192.168.40.1
- G0/0.60: 192.168.61.1
- G0/0.99: 192.168.91.1

### R3-BR2

- G0/0.50: 192.168.50.1
- G0/0.99: 192.168.92.1

Each subinterface is tied to its VLAN with `encapsulation dot1Q <id>`.

## Reused VLAN IDs

VLAN 60 exists at HQ and BR1, and VLAN 99 exists at all three sites. VLAN IDs only matter within a
Layer 2 domain, so the numbers can repeat; the IP subnets cannot. HQ printers use 192.168.60.0/24,
BR1 printers use 192.168.61.0/24, and each site has its own management subnet.

## Evidence

- `04_Switch_Configurations/SW1_VLAN_Database.png` to `SW5_VLAN_Database.png`
- `04_Switch_Configurations/SW1_Trunks.png` to `SW5_Trunks.png`
