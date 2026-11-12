# Server port assigned to the wrong VLAN

## When it surfaced

While setting the six Sales PCs to obtain their addresses automatically, after the server's static
address and all four DHCP pools had been configured. None of the six received a lease.

## Symptom

Every PC at every site came back with a 169.254.x.x address, so no site could obtain addressing. From
the R1-HQ console the branch links and the printer VLAN still answered pings, but pinging 192.168.30.10
did not, so routing was intact and the server was missing from its own subnet.

## Troubleshooting

### Layer 1 — Physical

The switch port serving the server was up, the server's NIC reported connected, and no errors were
counted on the interface. Cabling and hardware were cleared quickly.

### Layer 2 — Data link

`show vlan brief` on SW3-HQ-SERVICES showed FastEthernet0/1 sitting in VLAN 60 instead of VLAN 30.
`show mac address-table interface fa0/1` listed the server's MAC address under VLAN 60 in the switch's
own table, which confirmed where the port was putting its frames.

The server's address never changed. It is set statically at 192.168.30.10 and stayed that way; the
switch moved which VLAN its frames belonged to, so a host with a server-network address was now sitting
in the printer network, where its own gateway does not exist. On the server's own console it could ping
its own address and nothing else, its configured gateway 192.168.30.254 included.

### Layer 3 — Network

With no lease to work from, I gave one Sales PC a static address so the layers above addressing could be
tested. It pinged 192.168.10.1 and reached 198.51.100.10 through NAT, so routing and translation were
healthy, but it could not reach 192.168.30.10. `sh ip arp` on R1-HQ showed no entry for that address on
G0/0.30 even after the ping attempt, so the router was asking for it in the right VLAN and getting no
reply. The gateway itself was addressed correctly on G0/0.30, which left a host sitting in the wrong
broadcast domain rather than a missing route.

### Layer 4 to 7 — Transport and application

DNS, HTTP and DHCP all run on that one host, so all three were unreachable together. Every site losing
DHCP at the same moment pointed at the server's own path rather than at any single relay, which is what
sent me to the switch rather than to the server.

## Fix

```
SW3-HQ-SERVICES>en
SW3-HQ-SERVICES#config t
SW3-HQ-SERVICES(config)#int fa0/1
SW3-HQ-SERVICES(config-if)#sw mode access
SW3-HQ-SERVICES(config-if)#sw access vlan 30
SW3-HQ-SERVICES(config-if)#end
SW3-HQ-SERVICES#wr
Building configuration...
[OK]
```

The Sales PCs then took 192.168.10.x leases and the intranet loaded by name from one of them. `sh vlan
br` listed Fa0/1 under VLAN 30 again.

## What it taught me

Several services failing together on one host is one fault, not several. The port assignment is the
first place to look, because it is a single line that takes the whole host out of its own network while
leaving everything else untouched.
