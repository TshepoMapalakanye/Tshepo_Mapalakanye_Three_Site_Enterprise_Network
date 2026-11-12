# 04 — Switch Configuration Summary

What was configured on each switch and which screenshot proves it.

## VLANs and access ports

### SW1-HQ-CORE

- VLANs: 10, 20, 30, 60, 99, 999
- Management PC: FastEthernet0/1 in VLAN 99
- Trunks: to R1-HQ, SW2-HQ-USERS and SW3-HQ-SERVICES

### SW2-HQ-USERS

- Sales: FastEthernet0/1 to 0/6 in VLAN 10
- IT: FastEthernet0/7 to 0/9 in VLAN 20

### SW3-HQ-SERVICES

- Server: FastEthernet0/1 in VLAN 30
- Printer: FastEthernet0/2 in VLAN 60

### SW4-BR1

- Users: FastEthernet0/1 to 0/3 in VLAN 40
- Printer: FastEthernet0/4 in VLAN 60

### SW5-BR2

- Users: FastEthernet0/1 and 0/2 in VLAN 50

The user VLANs lease from 100-address scopes between .100 and .199, so no endpoint holds an address
above .199 unless it was set by hand.

## Trunks

- SW1 to SW2, SW1 to SW3, SW1 to R1-HQ, SW4 to R2-BR1 and SW5 to R3-BR2
- Explicit allowed VLAN list on every trunk
- Native VLAN 999 on every trunk, unused and with no address

A mismatch here caused the one real fault in this build, recorded in
`05_Troubleshooting/Troubleshooting_Log.md`.

## Spanning tree

- SW1-HQ-CORE is primary root for the HQ VLANs, set by priority
- PortFast on endpoint-facing ports
- BPDU Guard on the same ports, so an unexpected switch shuts the port
- Each branch switch sits alone in its own Layer 2 domain, so its root needs no tuning

## Port security

- Enabled on endpoint-facing ports on SW2, SW3, SW4 and SW5
- Sticky learning records each connected MAC address
- Maximum secure addresses and a violation mode configured per port
- Checked with `show port-security address`

All four access switches carry the same policy on their in-use endpoint ports. SW2 has the most
endpoints of any switch, so its output shows the pattern across user ports.

No violation was triggered, so the documentation records the configured mode rather than a test.

## Unused ports

Every unused access port is shut down.

## Management addressing

- SW1-HQ-CORE: 192.168.99.11
- SW2-HQ-USERS: 192.168.99.12
- SW3-HQ-SERVICES: 192.168.99.13
- SW4-BR1: 192.168.91.11
- SW5-BR2: 192.168.92.11

The SVIs give each switch a management address only; inter-VLAN routing stays on the routers.

## Baseline on all five switches

Hostname, banner, enable secret, local administrator account, SSH with RSA keys and a domain name,
VTY lines accepting SSH only, management SVI, configuration saved to startup.

## Evidence

- `SW1_VLAN_Database.png` to `SW5_VLAN_Database.png`
- `SW1_Trunks.png` to `SW5_Trunks.png`
- `SW1_HQ_STP_Root.png`
- `SW2_Port_Security.png`
