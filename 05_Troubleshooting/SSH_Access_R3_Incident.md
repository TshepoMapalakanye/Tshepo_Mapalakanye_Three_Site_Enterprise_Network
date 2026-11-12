# SSH access lost on R3-BR2

## When it surfaced

I was applying the same baseline configuration to R3-BR2 that the other routers already had, ending
with `line vty 0 4` and `transport input ssh`. I had not generated an RSA key pair on this router yet,
so the lines had nothing to authenticate with.

## Symptom

SSH from HQ-MANAGEMENT-PC was refused outright, with no password prompt, while the router continued to
answer pings and forward traffic for both of its VLANs. Users noticed nothing; only remote
administration was affected.

## Troubleshooting

### Layer 1 and 2 — Physical and data link

The path was healthy. Pings to other devices on the same segment answered, and the router's own
interfaces were up, so nothing below Layer 3 was involved.

### Layer 3 — Network

The router responded to pings, so it was reachable and the fault sat entirely above the network layer.

### Layer 4 to 7 — Transport and application

`ssh -l admin` failed at connection setup rather than at authentication. A refusal at that point means
nothing is listening for that protocol on that line, which is different from a rejected password.

With no remote path in, the only way back was the console in Packet Tracer. From the console,
`show ip ssh` reported SSH as disabled and `show crypto key mypubkey rsa` showed no keys at all. The
VTY lines were requiring a transport the router had no key pair to support.

## Fix

```
R3-BR2>en
R3-BR2#config t
R3-BR2(config)#crypto key generate rsa
How many bits in the modulus [512]: 2048
R3-BR2(config)#line vty 0 4
R3-BR2(config-line)#transport input ssh
R3-BR2(config-line)#end
R3-BR2#wr
Building configuration...
[OK]
```

I generated the key pair with a 2048-bit modulus, which is large enough for SSH version 2, then re-entered
the VTY lines to confirm they were still set to accept SSH only. `sh ip ssh` reported SSH enabled and a
session from the management PC authenticated normally.

## What it taught me

`transport input ssh` assumes a key pair already exists. The order matters: generate the keys, then
restrict the transport. Keeping console access is not optional, because a router answering pings tells
you nothing about whether its management protocol works.
