# SR-MPLS Entropy Label and ECMP Troubleshooting Checklist

Use this checklist when routes and labels are correct but traffic is uneven across ECMP paths or LAG members.

## Key concepts

- The Entropy Label Indicator (ELI) is reserved label 7.
- The Entropy Label (EL) follows the ELI and carries flow entropy for load-balancing decisions.
- Transit devices can hash on the EL without inspecting the customer payload.
- An entropy label does not encrypt traffic and does not replace the transport or service label.

## 1. Confirm the forwarding topology

- [ ] Verify all expected ECMP next hops are installed.
- [ ] Verify every LAG member is operational and forwarding.
- [ ] Confirm link metrics and adjacencies are consistent.
- [ ] Check that no policy, affinity, or protection state removes a path.
- [ ] Confirm forward and reverse paths separately.

## 2. Verify entropy-label capability

- [ ] Confirm ingress, transit, and egress platforms support entropy labels.
- [ ] Check the applicable IGP/BGP capability advertisement and local configuration.
- [ ] Verify the ingress node is allowed to impose ELI/EL.
- [ ] Confirm there is no interoperability issue between software releases or vendors.

## 3. Inspect the imposed label stack

- [ ] Capture a packet at the ingress PE or edge LSR.
- [ ] Locate ELI (label 7) and the following entropy label.
- [ ] Verify transport, service, ELI, and EL ordering.
- [ ] Compare the programmed stack with the observed packet.
- [ ] Validate maximum supported label depth and encapsulation overhead.

## 4. Validate hashing behavior

- [ ] Identify which MPLS fields the platform uses for ECMP and LAG hashing.
- [ ] Confirm the device actually includes the entropy label in its hash.
- [ ] Verify packets from one flow remain on a stable member.
- [ ] Generate multiple flows with different five-tuples and compare distribution.
- [ ] Test both directions because hashing may be asymmetric.

## 5. Compare counters

- [ ] Record packets and bytes on every ECMP path or LAG member.
- [ ] Check interface errors, discards, queue drops, and policer drops.
- [ ] Look for one hot member while parallel members remain idle.
- [ ] Compare ingress and egress counters to locate the imbalance point.
- [ ] Check hardware forwarding counters, not only control-plane statistics.

## 6. Controlled test matrix

| Test | Change | Expected result |
|---|---|---|
| Single flow | Fixed source/destination and ports | Stable path selection |
| Multiple flows | Vary the five-tuple | Distribution across available members |
| Reverse traffic | Swap source and destination | May use a different hash/path |
| Entropy variation | Generate different EL values | Distribution should change predictably |
| EL disabled | Remove ELI/EL where safe | Compare fallback MPLS hashing behavior |

## 7. Hardware and software checks

- [ ] Confirm the line card or NPU supports EL-based hashing.
- [ ] Check platform limits for label depth and ECMP width.
- [ ] Review known defects for the running software release.
- [ ] Verify QoS, ACL, or service-policy processing is not forcing a path or dropping traffic.
- [ ] Confirm recent configuration or upgrade changes.

## 8. Evidence to collect

- Routing and ECMP next-hop output
- MPLS forwarding/LFIB output
- Imposed label-stack output
- ELI/EL capability and configuration
- Per-member packet, byte, error, and drop counters
- Packet captures from ingress and the suspected imbalance point
- Test-flow parameters and timestamps

## Quick diagnosis

If one member is consistently hot while routing and labels are correct, investigate hashing and entropy-label processing before changing the control plane. If ELI/EL is present but distribution is still poor, compare the platform hash inputs, per-member counters, traffic symmetry, and hardware limits.

## References

- RFC 6790 — The Use of Entropy Labels in MPLS Forwarding
- RFC 8662 — Entropy Label for Source Packet Routing in Networking (SPRING) Tunnels
