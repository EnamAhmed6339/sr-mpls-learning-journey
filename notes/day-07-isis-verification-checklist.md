# Day 7 — IS-IS Segment Routing Verification Checklist

A practical, three-layer check for an IOS XR IS-IS core after enabling SR-MPLS. This is a teaching reference based on standard IOS XR behaviour; it is not presented as captured lab output.

## Configuration prerequisites

- `metric-style wide` is active wherever SR sub-TLVs must be advertised.
- `segment-routing mpls` is enabled under the intended IS-IS address family.
- Each loopback has a unique `prefix-sid index`.
- Loopbacks are passive in IS-IS unless an adjacency is intentionally required.
- The SRGB is documented on every router; matching blocks simplify operations but are not required for forwarding.

## Three-layer verification

### 1. Advertisement

```text
show isis database verbose
```

Confirm the router capability/SRGB information and the Prefix-SID sub-TLV for each expected loopback.

### 2. Label computation

```text
show isis segment-routing label table
```

Confirm each SID index resolves to the expected local label:

```text
local label = local SRGB base + SID index
```

### 3. Forwarding

```text
show mpls forwarding
```

Confirm the LFIB contains the expected `SR Pfx` entries and the correct push, swap or pop action.

## Symptom-to-check map

| Symptom | First check | Likely direction |
| --- | --- | --- |
| IS-IS adjacency is up, but no Prefix-SID appears | `show isis database verbose` | Wide metrics or SR advertisement configuration |
| Prefix-SID is present, but the label is unexpected | SRGB and SID index | Local SRGB difference or duplicate/wrong index |
| Label table is correct, but traffic is not label switched | `show mpls forwarding` | LFIB programming or next-hop resolution |
| Only one loopback is missing | Loopback IS-IS and Prefix-SID configuration | Passive/interface or prefix advertisement issue |
| Multiple routers resolve the same index for different prefixes | Audit all Prefix-SID indexes | Duplicate index |

## Operational rule

Verify advertisement, then label computation, then forwarding. A healthy adjacency proves reachability between IS-IS neighbors; it does not prove that SR information was advertised or programmed.

Related lesson: [Day 7 — Configuring SR with IS-IS](https://youtu.be/9B0sjZa6Ono)
