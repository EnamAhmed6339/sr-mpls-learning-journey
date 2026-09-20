# Day 6 — SR-MPLS LFIB Packet Walk

This note follows one packet from ingress to egress and separates the three decisions that make SR-MPLS forwarding work:

1. The SID expresses the destination or policy intent.
2. The IGP selects the next hop.
3. The LFIB applies the label operation.

## Example topology

`R1 → R2 → R3 → R4`

R4 advertises a global Prefix/Node SID index. Each router learns the SID and the SRGB information through the IGP.

| Hop | Control-plane input | Forwarding action |
| --- | --- | --- |
| R1 — ingress | Route to R4, SID index, and R2's SRGB | Push the label R2 expects for the SID |
| R2 — transit | Local incoming label and route toward R4 | Swap to the label R3 expects |
| R3 — penultimate | Local incoming label and PHP behavior | Pop the transport label |
| R4 — egress | Destination IP lookup | Deliver or continue IP forwarding |

## Why the label number may not change

When every router uses the same SRGB, the incoming and outgoing numeric labels can be identical. The LFIB lookup still occurs; identical numbers do not mean the router skipped the swap decision.

When SRGB ranges differ, each upstream router derives the outgoing label from the next hop's advertised SRGB and the global SID index. The numeric label can therefore change at each hop while the SID intent remains the same.

## PHP and explicit-null

Penultimate-hop popping removes the transport label before the packet reaches the egress router. Explicit-null keeps a label to the final hop when preserving MPLS QoS or TTL treatment is required.

## Verification workflow

Use the equivalent commands for your platform and software release:

1. Confirm the destination route and next hop.
2. Confirm the Prefix-SID and SRGB advertisements in the IGP database.
3. Verify the local and outgoing LFIB entries.
4. Check label counters while sending test traffic.
5. Confirm whether PHP or explicit-null is expected.

Useful IOS XR starting points:

`show route <destination-prefix>`

`show ospf segment-routing prefix-sid-map`

`show mpls forwarding`

`show cef <destination-prefix> detail`

## Troubleshooting order

If the expected label is missing, check in this order:

1. SID and SRGB advertisement
2. IGP reachability and selected next hop
3. LFIB programming
4. PHP or explicit-null behavior
5. Data-plane counters and packet capture

The key lesson is simple: **the SID defines intent, the IGP chooses the path, and the LFIB performs the hop-by-hop action.**
