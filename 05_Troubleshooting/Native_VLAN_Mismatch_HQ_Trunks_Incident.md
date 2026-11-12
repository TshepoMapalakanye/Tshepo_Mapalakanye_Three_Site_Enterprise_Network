# Native VLAN mismatch on the HQ trunks

## When it surfaced

I had configured FastEthernet0/23 on SW1-HQ-CORE as a trunk with native VLAN 999, then cabled it to
FastEthernet0/23 on SW2-HQ-USERS, which was still running its default configuration. The warning
appeared on the SW1 console the moment the link came up.

## Symptom

The console reported:

```
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/23 (999),
with SW2-HQ-USERS FastEthernet0/23 (1).
```

Nothing else looked wrong. The link was up, user traffic crossing the trunk worked, and no endpoint
reported a problem. The two switches simply disagreed about which VLAN untagged frames belonged to.

## Troubleshooting

### Layer 1 — Physical

Both ends of the link were up and FastEthernet0/23 showed no errors, so the cable and the ports were
not the problem. This was cleared first and took no more than checking interface status at each end.

### Layer 2 — Data link

`show interfaces trunk` on SW1-HQ-CORE reported native VLAN 999 on FastEthernet0/23. The same command
on SW2-HQ-USERS reported native VLAN 1 on its end of the same link. The allowed VLAN lists and the
trunk state matched on both sides, so the native VLAN was the only difference.

Where the trunk output did not confirm the setting, I checked the interface configuration directly and
found the same thing: the core switch had been given the value, the downstream switch had not.

### Layer 3 and above

There was no reason to test these. The trunk was forwarding tagged traffic, so gateways and services
were reachable and any failure here would have been a separate fault. Untagged control traffic was the
only traffic at risk, which is why nothing appeared broken from an endpoint.

## Fix

I set the native VLAN on both ends rather than only the end that complained, and applied an explicit
allowed list on the same interfaces.

```
SW1-HQ-CORE>en
SW1-HQ-CORE#config t
SW1-HQ-CORE(config)#int fa0/23
SW1-HQ-CORE(config-if)#sw mode trunk
SW1-HQ-CORE(config-if)#sw trunk native vlan 999
SW1-HQ-CORE(config-if)#sw trunk allowed vlan 10,20,30,60,99
SW1-HQ-CORE(config-if)#end
SW1-HQ-CORE#wr
Building configuration...
[OK]
```

The same block was run on fa0/23 of SW2-HQ-USERS, and then on both ends of the SW1 to SW3 trunk
(SW1 fa0/24 and SW3 fa0/23), which had been configured the same way and had not reported anything of its
own yet.

Verification was `sh int trunk` on each switch showing native VLAN 999 at both ends, the warning no
longer appearing, and DHCP leases succeeding for the user VLANs that cross that trunk.

## What it taught me

A trunk is one object with two ends. Setting the core switch does not set the access switch, and a
mismatch does not necessarily drop the link, so it has to be found rather than waited for. Reading the
warning properly saved time: it named both interfaces and both values, so the work was confirming
rather than hunting.
