# DHCP handing out an unreachable default gateway

## When it surfaced

I had set the six Sales PCs to obtain their addresses automatically and was renewing their leases one at
a time to confirm the Sales pool worked. I renewed the first PC, then pinged its default gateway from
it, and the ping failed.

## Symptom

One PC came back with a legitimate 192.168.10.x address and mask, could reach the other PCs on its own
subnet, and could not reach anything outside it. The address looked correct, which is what made the
fault easy to miss.

## Troubleshooting

### Layer 1 — Physical

The PC's link was up, the switch port was up, and neither side had errors counted against it.

### Layer 2 — Data link

The PC's MAC address was learned in VLAN 10 on SW2-HQ-USERS, so the port was in the correct VLAN and
the frame was not being placed anywhere unexpected.

### Layer 3 — Network

`ipconfig` showed the default gateway as 192.168.10.254. Pinging another Sales PC succeeded, which
proved the local segment, the address and the mask were all sound. Ping 192.168.20.1 failed, which is
what a host with an unusable gateway looks like: it reaches its own subnet and nothing beyond it.
Pinging the address it had been given, 192.168.10.254, also failed, so the value the client was
trusting did not exist on the network.

The addressing plan sets 192.168.10.1 as the gateway on R1-HQ G0/0.10, so the client had been given a
value that was never part of the design.

### Layer 4 to 7 — Transport and application

Everything above this layer fails by design once the first hop is wrong, so no service test was worth
running until the gateway was corrected.

## Fix

When I created the Sales pool I typed 192.168.10.254 into the Default Gateway field instead of
192.168.10.1, and DHCP hands that field to every client unchanged, so the slip reached every PC in the
VLAN. The wrong value sat in the pool on HQ-SERVER01, not on a router, so the correction was made there:

- Device: HQ-SERVER01
- Tab: Services, then DHCP
- Pool Name: Sales
- Default Gateway: 192.168.10.1, which replaces 192.168.10.254
- Subnet Mask: 255.255.255.0
- Start IP Address: 192.168.10.100
- Maximum Users: 100

I changed the gateway field and left the rest of the pool as it was. Every client then had to be given
the new value, because a lease already held is not rewritten:

```
C:\> ipconfig /release
C:\> ipconfig /renew
```

Verification was `ipconfig` on a renewed PC showing `192.168.10.1`, a reply to a ping to the gateway,
and the intranet loading by name.

## What it taught me

A successful lease proves the client was answered, not that it was answered correctly. Reading every
field of the lease rather than only the address is what found this, and renewing every client afterwards
matters because each one would otherwise have kept the bad value for the rest of its lease time.
