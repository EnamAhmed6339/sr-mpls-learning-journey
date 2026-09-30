# SR-MPLS MTU Troubleshooting Checklist

Use this checklist when SR-MPLS routing and labels appear correct, small packets pass, but larger packets or applications fail.

## Why MTU changes with MPLS

Each MPLS shim header is **4 bytes**. The ingress router can impose more than one label, so calculate the complete encapsulation overhead instead of checking only the IP packet size.

| Encapsulation item | Typical overhead |
|---|---:|
| One MPLS label | 4 bytes |
| Two-label transport + service stack | 8 bytes |
| Entropy Label Indicator + Entropy Label | 8 bytes |
| 802.1Q VLAN tag | 4 bytes |
| QinQ outer tag | 4 additional bytes |
| GRE | 4+ bytes, depending on options |
| IPv4 outer header | 20+ bytes |
| IPv6 outer header | 40+ bytes |
| IPsec ESP | Variable; calculate from the selected mode and algorithms |

> Always validate the exact platform behavior and encapsulation combination used in the production path.

## Common symptoms

- Small pings succeed while larger pings fail.
- TCP sessions establish but transfers stall or reset.
- Routing tables and LFIB entries look correct, yet selected applications fail.
- Problems begin only after adding a service label, SR policy, GRE, IPsec, VLAN, or pseudowire.
- One direction works while the reverse path fails.
- Interface counters show giants, discards, or hardware drops.
- ICMP Packet Too Big or Fragmentation Needed messages are filtered, creating a PMTUD black hole.

## Fault-isolation workflow

### 1. Establish the failure boundary

- Test the customer-facing path and the core path separately.
- Start with a small packet and increase the payload in controlled steps.
- Repeat with the Don't Fragment bit set.
- Test both directions because forwarding paths and label stacks can differ.
- Record the largest successful size and the smallest failing size.

### 2. Confirm the imposed label stack

At the ingress PE, verify:

- transport or Prefix-SID label;
- VPN, EVPN, or pseudowire service label;
- SR policy or Binding SID labels;
- Entropy Label Indicator and Entropy Label;
- Explicit Null behavior;
- any GRE, IPsec, VLAN, or QinQ encapsulation.

Do not assume that the control-plane route displays the complete forwarding stack. Check the programmed forwarding entry.

### 3. Calculate required MTU

Use this simple model:

`required frame size = customer packet + MPLS labels + other encapsulation + Layer-2 overhead`

Example: a 1500-byte IP packet with two MPLS labels requires 1508 bytes before accounting for Ethernet framing and any VLAN tags.

### 4. Verify MTU hop by hop

For every core, access, and interconnect link, compare:

- physical interface MTU;
- IP MTU;
- MPLS MTU, where the platform exposes it separately;
- subinterface and bundle-member settings;
- provider handoff limits;
- tunnel or security-policy overhead;
- hardware forwarding limits.

A single lower-MTU hop can create an end-to-end black hole.

### 5. Inspect counters and the data plane

Look for:

- input and output giants;
- oversize-frame counters;
- output drops and queue discards;
- NP, NPU, or line-card drop reasons;
- tunnel encapsulation failures;
- ICMP error generation and filtering;
- LFIB push, swap, pop, and disposition actions.

Clear counters only when operational policy permits, reproduce the issue once, and compare the deltas.

### 6. Validate PMTUD

- Confirm that ICMP Fragmentation Needed or Packet Too Big messages can return to the source.
- Check firewalls, ACLs, and control-plane policing.
- Compare behavior with and without DF.
- For TCP, inspect the negotiated MSS and consider whether MSS adjustment is appropriate at the edge.

MSS adjustment can mitigate TCP symptoms, but it does not replace correcting an incorrect transport MTU.

## Test matrix

| Test | Expected result | What failure suggests |
|---|---|---|
| Small ping, no DF | Success | Basic reachability exists |
| Large ping, DF set | Success up to designed PMTU | MTU or PMTUD issue if it fails early |
| Core loopback-to-loopback test | Success | Core transport MTU is adequate |
| CE-to-CE service test | Success | Access, service label, or handoff issue if core passes |
| Forward and reverse tests | Same designed PMTU | Asymmetric path or label-stack difference |
| Counter check during reproduction | No oversize/drop increments | Specific dropping interface or forwarding resource |

## Vendor-neutral verification checklist

- [ ] IGP route to the next hop is present.
- [ ] Prefix-SID and SRGB information are valid.
- [ ] The LFIB contains the expected outgoing label action.
- [ ] The imposed label stack matches the intended service and policy.
- [ ] Interface, IP, and MPLS MTU values are consistent end to end.
- [ ] Bundle members and subinterfaces use compatible MTU values.
- [ ] PMTUD ICMP messages are not blocked.
- [ ] VLAN, pseudowire, GRE, and IPsec overhead are included.
- [ ] Hardware drop counters remain clean during the test.
- [ ] Both traffic directions have been validated.

## Evidence to capture

For a reproducible incident record, save:

1. topology and traffic direction;
2. working and failing packet sizes;
3. DF behavior;
4. ingress label stack;
5. per-hop MTU values;
6. relevant interface and hardware counters;
7. packet capture showing the last successful hop;
8. the configuration change or remediation;
9. the post-change verification results.

## Key lesson

A healthy IGP and a correct LFIB do not guarantee that every packet size can cross the path. When small traffic succeeds and larger traffic fails, validate the entire encapsulation stack and end-to-end MTU before changing routing.
