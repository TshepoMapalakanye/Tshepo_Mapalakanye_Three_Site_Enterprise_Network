# One VLAN removed from a trunk's allowed list

## When it surfaced

When I added the allowed VLAN lists to the HQ trunks at the start, I typed
`switchport trunk allowed vlan 20,30,60,99` on FastEthernet0/23 of SW2-HQ-USERS and forgot to include
VLAN 10. The lists were entered by hand on each trunk, and I moved on to the next interface without
checking that one against the VLAN plan.

## Symptom

Six Sales PCs lost everything, including their own gateway. Three IT PCs on the same switch were
untouched. A fault that takes out one department and leaves its neighbour alone is worth reading
carefully before changing anything.

## Troubleshooting

### Layer 1 — Physical

Both ends of the trunk were up, so cabling was not involved.

### Layer 2 — Data link

`show interfaces trunk` on SW2-HQ-USERS showed an allowed list without VLAN 10. The same command on
SW1-HQ-CORE showed the full list on the matching interface, so the two ends disagreed about what the
trunk was permitted to carry.

VLAN 20 was still in the list and still crossing the trunk, which is exactly why the IT PCs on the same
switch never noticed anything. VLAN 60 and VLAN 99 were unaffected for the same reason. Running
`show vlan brief` on SW2 confirmed VLAN 10 still existed locally, so the VLAN had not been removed,
only excluded from the trunk.

### Layer 3 — Network

An IT PC could reach the gateway and a Sales PC could not, over the same physical path. That ruled out
routing before any route command was run, because the only difference between the two was which tag the
frame carried.

## Fix

```
SW2-HQ-USERS>en
SW2-HQ-USERS#config t
SW2-HQ-USERS(config)#int fa0/23
SW2-HQ-USERS(config-if)#sw trunk allowed vlan 10,20,30,60,99
SW2-HQ-USERS(config-if)#end
SW2-HQ-USERS#wr
Building configuration...
[OK]
```

The Sales PCs reached their gateway and the internet again, and `sh int trunk` showed matching lists on
both switches.

## What it taught me

Compare both ends of a trunk, not one. An allowed list is easy to trim by accident while tidying other
VLANs on the same interface, and the result looks like a routing or gateway failure from the endpoint
even though nothing above Layer 2 has changed.
