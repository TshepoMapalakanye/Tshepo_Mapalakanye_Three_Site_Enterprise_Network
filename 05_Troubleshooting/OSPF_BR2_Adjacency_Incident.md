# BR2 lost its OSPF adjacency to HQ

## When it surfaced

When I set up OSPF on R1-HQ I used `passive-interface default` and then exempted the two WAN links one
at a time. I exempted Serial0/0/0 and tested BR2, but forgot to exempt Serial0/0/1 before moving on.

## Symptom

BR1 stayed normal. BR2 lost every HQ and every internet route. No error appeared on any console, and
nothing changed immediately: the loss took about forty seconds, which is the dead interval expiring.

## Troubleshooting

### Layer 1 — Physical

`show ip interface brief` on R1-HQ showed Serial0/0/1 up and up, so the link was healthy and this was
not a cabling or interface problem.

### Layer 2 — Data link

Not applicable on a routed serial link.

### Layer 3 — Network

`show ip ospf neighbor` on R1-HQ listed one neighbour instead of two, which narrowed the fault to a
single adjacency rather than a routing policy problem.

`show ip ospf interface Serial0/0/1` reported the interface as passive, which explained the missing
hellos. `show running-config | section router ospf` showed that the exemption for that interface had
never been re-added, so it was inheriting the default.

On R3-BR2, `show ip route` had collapsed to connected routes. That confirmed the far end was healthy
and had simply not been heard from.

### Layer 4 to 7 — Transport and application

Nothing could be tested above the network layer until the routes returned.

## Fix

```
R1-HQ>en
R1-HQ#config t
R1-HQ(config)#router ospf 1
R1-HQ(config-router)#no passive-interface s0/0/1
R1-HQ(config-router)#end
R1-HQ#wr
Building configuration...
[OK]
```

Both branch neighbours returned to FULL on R1-HQ, and R3's routing table repopulated with the HQ networks
and the default route. `sh ip ospf nei` on R1-HQ and `sh ip route` on R3-BR2 confirmed it from each end.

## What it taught me

`passive-interface default` fails quietly and slowly: it looks correct in the running config, produces
no log message and takes the full dead interval to show itself. The neighbour table is the cheapest
place to find out, and an adjacency count is easier to check than a routing table.
