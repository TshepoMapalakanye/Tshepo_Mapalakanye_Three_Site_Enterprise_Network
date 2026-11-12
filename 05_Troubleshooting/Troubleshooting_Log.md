# 05 — Troubleshooting Log

Seven faults came up while building this network. Each one has a full account in this folder, covering
how it surfaced, the order I worked it in and what corrected it. This page is the index.

## Native VLAN mismatch on the HQ trunks

The two ends of the SW1 to SW2 trunk disagreed about the native VLAN. The core switch used 999 while
the downstream switch still had the default VLAN 1. Tagged user traffic kept passing, so no endpoint
reported a fault and only the console warning showed it.

Fix: set native VLAN 999 and an explicit allowed list on both ends of FastEthernet0/23, repeated the
same values on the SW1 to SW3 trunk, and confirmed with `show interfaces trunk` that both ends matched.

Full account: `Native_VLAN_Mismatch_HQ_Trunks_Incident.md`

## Server port assigned to the wrong VLAN

The HQ server's port on SW3 was still in VLAN 60, so a host holding a server-network address was
sitting in the printer network, where its own gateway does not exist. No PC at any site could obtain an
address, and DNS and HTTP were unreachable along with it.

Fix: `switchport access vlan 30` on FastEthernet0/1 of SW3-HQ-SERVICES, saved, then confirmed the Sales
PCs took leases and the intranet loaded by name.

Full account: `Server_Port_Wrong_VLAN_Incident.md`

## DHCP handing out an unreachable default gateway

The Sales pool on the central server carried `192.168.10.254` as its default router, so every renewed
client held a valid address it could not route from.

Fix: corrected the pool gateway to `192.168.10.1` and renewed all six Sales PCs so none kept the bad
lease.

Full account: `DHCP_Gateway_Address_Incident.md`

## One VLAN removed from a trunk's allowed list

The allowed list entered on the SW2 uplink left VLAN 10 out, so the Sales VLAN stopped crossing the
trunk. Six Sales PCs lost their gateway while the IT PCs on the same switch carried on working.

Fix: restored `switchport trunk allowed vlan 10,20,30,60,99` on FastEthernet0/23 and confirmed the
list matched on both switches.

Full account: `Trunk_Allowed_VLAN_List_Incident.md`

## BR2 lost its OSPF adjacency to HQ

A passive-interface change on R1-HQ left the serial link to R3-BR2 passive, so no hellos were sent on
it. BR2 lost every HQ and internet route about forty seconds later, with no error on any console.

Fix: re-added `no passive-interface Serial0/0/1` so the interface was exempt from the default, then
confirmed both branch neighbours returned to FULL.

Full account: `OSPF_BR2_Adjacency_Incident.md`

## Branch port disabled by a port-security violation

A second device placed behind one SW4-BR1 port presented an extra MAC address, which the port was
configured to refuse. The switch put the port into err-disabled and both devices behind it went
offline.

Fix: removed the extra device, then `shutdown` and `no shutdown` on FastEthernet0/1. The sticky entry
returned and the port carried traffic again. Automatic err-disable recovery was left off.

Full account: `Port_Security_Errdisable_Incident.md`

## SSH access lost on R3-BR2

The VTY lines on R3-BR2 were set to accept SSH only before the RSA key pair existed, so there was
nothing on the router to authenticate a session. SSH was refused while the router still answered
pings and forwarded traffic.

Fix: consoled in, ran `crypto key generate rsa` with a 2048-bit modulus, confirmed `show ip ssh`
reported SSH enabled, and saved.

Full account: `SSH_Access_R3_Incident.md`
