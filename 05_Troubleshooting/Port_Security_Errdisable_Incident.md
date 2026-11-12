# Branch port disabled by a port-security violation

## When it surfaced

I had just configured sticky learning with a maximum of one address on the SW4-BR1 user ports and was
testing that setting by connecting a second device behind FastEthernet0/1.

## Symptom

Both devices behind that port went offline. The other branch ports kept working, and the port light
stayed on, which is why the fault looked like nothing at all until the switch was asked directly.

## Troubleshooting

### Layer 1 — Physical

The interface was up and the light was on, so nothing suggested a cabling fault. This is the layer that
delays the diagnosis, because a dead port normally shows here and this one did not.

### Layer 2 — Data link

`show interfaces status` read err-disabled on FastEthernet0/1, which says the switch took the port down
deliberately rather than losing the link.

`show port-security interface fa0/1` showed the violation count, the shutdown violation mode and the
maximum of one secure address. `show port-security address` still held the original sticky MAC address,
so the port had learned the right device first and then refused a second one.

### Layer 3 and above

Not applicable. The port was administratively disabled, so nothing above Layer 2 could be tested until
it was recovered.

## Fix

I removed the extra device, then recovered the port:

```
SW4-BR1>en
SW4-BR1#config t
SW4-BR1(config)#int fa0/1
SW4-BR1(config-if)#shut
SW4-BR1(config-if)#no shut
SW4-BR1(config-if)#end
SW4-BR1#wr
Building configuration...
[OK]
```

The sticky entry returned and traffic resumed. Automatic err-disable recovery was deliberately left
off: it is reasonable on a user port where a device swap is common, and wrong on an uplink where an
automatic recovery would hide a real problem.

## What it taught me

Port security working correctly looks like a dead port. Err-disabled means the switch protected the
network, so the work is to recover the port and keep the control, not to disable the feature to make
the port come back.
