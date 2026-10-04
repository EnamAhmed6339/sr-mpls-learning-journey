# SR-MPLS Binding SID Troubleshooting Checklist

## Purpose

A Binding SID (BSID) represents an entire Segment Routing policy with one locally significant SID or MPLS label. An upstream node can steer traffic toward the BSID owner without carrying the policy's full segment list. The owner resolves the BSID into the active candidate path and imposes the expanded list.

This checklist provides a vendor-neutral workflow for proving that a BSID is allocated, advertised or referenced correctly, programmed in the data plane, expanded within platform limits, and reoptimized safely after a path failure.

## Core concepts

- **BSID owner:** The router that allocates the BSID and resolves it into an SR policy.
- **Candidate path:** One possible path for the policy, selected according to preference and validity.
- **Segment list:** The ordered Prefix-SIDs, Adj-SIDs or other instructions imposed by the BSID owner.
- **Recursion:** The process of resolving the BSID to a policy, the policy to a segment list, and each segment to a reachable next hop.
- **Maximum SID Depth (MSD):** The platform limit on the number of SIDs or labels that can be imposed.
- **Locally significant label:** A label whose meaning is defined on the node that owns and programs it.

## 1. Capture the baseline

Record the following before changing the policy:

- Policy name, color, endpoint and head-end
- BSID value and owner
- Active and standby candidate paths
- Candidate-path preference and origin
- Segment-list contents and weights
- Route to the policy endpoint
- Reachability to every Prefix-SID and Adj-SID
- SRGB and SRLB ranges
- RIB, LFIB and hardware entries for the BSID
- Interface counters, MPLS drops and policy counters
- Platform MSD, MTU and label-stack limits
- Current latency, loss and selected path

## 2. Confirm BSID ownership and allocation

1. Verify the BSID is allocated on the intended head-end.
2. Confirm the label does not collide with another local label or reserved range.
3. Check whether the BSID is static, dynamic or controller-assigned.
4. Verify that the control plane reports the expected owner.
5. Confirm upstream policies or routes steer the packet to that owner before the BSID is interpreted.
6. If the BSID is advertised externally, verify that the advertisement points to the correct node and policy.

A BSID is locally meaningful. Another router must first deliver the labeled packet to the owner that programmed its interpretation.

## 3. Verify policy identity and state

Confirm that the BSID maps to the intended policy attributes:

| Attribute | Expected evidence |
|---|---|
| Head-end | Correct router owns the policy and BSID |
| Endpoint | Intended destination address |
| Color or intent | Correct service or traffic-engineering objective |
| Candidate path | Highest-preference valid path is active |
| Segment list | Intended ordered set of segments |
| Operational state | Up with a programmed forwarding entry |

Check for a policy that is administratively up but operationally down, unresolved, inactive or rejected.

## 4. Validate candidate-path selection

- List every candidate path and its preference.
- Confirm the expected candidate is valid and active.
- Check controller, BGP, PCEP, configuration or local policy origin.
- Identify why any higher-preference path is invalid.
- Verify constraints such as affinity, metric, bandwidth, disjointness and SRLG.
- Confirm the endpoint and color resolve consistently.
- Check whether a stale controller path or withdrawn segment list remains referenced.

## 5. Decode the expanded segment list

For each segment in order:

1. Identify whether it is a Prefix-SID, Adj-SID, Anycast-SID or another BSID.
2. Map the SID or label to its owner and forwarding behavior.
3. Verify the next segment is reachable from the node that activates it.
4. Confirm Prefix-SIDs resolve through the intended IGP topology or algorithm.
5. Confirm Adj-SIDs refer to an up, correctly addressed local adjacency.
6. Check that nested BSIDs do not create a recursion loop.
7. Prove that the expanded path satisfies the intended constraints.

A syntactically valid segment list can still fail when a segment is unreachable, owned by the wrong node or recursively dependent on the policy itself.

## 6. Check label-stack depth

Count every label that can appear on the packet:

- Service or VPN label
- Transport label
- Binding SID
- Expanded policy segments
- TI-LFA repair labels
- Entropy Label Indicator and Entropy Label
- Explicit-null or other special-purpose labels

Compare the worst-case stack with the ingress and transit hardware limits. Check both the advertised node MSD and the real platform or line-card limit.

Also validate packet size. Each MPLS label adds four bytes, so a working small probe does not rule out an MTU failure on a deeper stack.

## 7. Verify forwarding-plane programming

Compare the control plane, LFIB and hardware table:

- The BSID label is present in the LFIB.
- The entry points to the intended active policy.
- The imposed segment list matches the control-plane view.
- The outgoing interface and next hop are correct.
- Hardware programming reports success.
- Policy and label counters increment when test traffic is sent.
- No unsupported action or label-depth error is recorded.

If the policy is up but counters remain at zero, verify traffic classification, steering and the route that selects the BSID.

## 8. Validate upstream steering

- Confirm the route, service, tunnel or policy that chooses the BSID.
- Verify the packet reaches the BSID owner with the expected top label.
- Check that intermediate routers transport the BSID without interpreting it incorrectly.
- Inspect packet captures at the upstream node and BSID owner.
- Verify that the owner pops or swaps the BSID and imposes the expanded segment list as designed.
- Confirm service labels remain intact beneath the transport policy.

## 9. Test failure and reoptimization

Use a controlled and reversible test.

### Before failure

- Confirm the preferred candidate path is active.
- Confirm standby or dynamic alternatives are valid.
- Record policy, LFIB, hardware and interface counters.
- Start timestamped probes and telemetry.

### During failure

1. Fail the intended link, node or policy segment.
2. Record failure-detection time.
3. Confirm the affected candidate path becomes invalid.
4. Verify the BSID stays programmed or transitions cleanly.
5. Confirm the next valid candidate becomes active.
6. Inspect the new expanded segment list.
7. Measure loss, latency and reoptimization time.
8. Check for blackholing, recursion loops and microloops.

### After restoration

- Confirm policy revertive or non-revertive behavior matches design.
- Verify the preferred candidate returns when appropriate.
- Confirm the LFIB and hardware entries are updated.
- Check that old segment-list entries and counters are not stale.

## 10. Evidence matrix

| Layer | Evidence | Healthy result | Failure indicator |
|---|---|---|---|
| Ownership | BSID allocation table | Correct head-end and unique label | Wrong owner or collision |
| Policy | Candidate-path detail | Intended path active | Down, unresolved or lower preference active |
| Recursion | Segment resolution | Every segment reachable | Recursive loop or unreachable segment |
| Labels | SRGB, SRLB and stack decode | Labels map to intended SIDs | Wrong mapping or missing SID |
| Depth | MSD and imposed stack | Within all platform limits | Stack exceeds hardware limit |
| Forwarding | LFIB and hardware table | Control and data planes agree | Entry missing or programming failed |
| Steering | Route and packet capture | Traffic reaches BSID owner | Traffic bypasses or misreaches owner |
| Resilience | Failure test | Clean candidate transition | Blackhole, loss or slow reoptimization |
| MTU | Sized probes and counters | Full-size traffic passes | Giants, drops or fragmentation issue |

## 11. Common failure patterns

- The BSID is configured on a different head-end than expected.
- Two local functions attempt to use the same label.
- A higher-preference candidate path is invalid, so an unintended path becomes active.
- A Prefix-SID is absent from the required IGP topology or algorithm.
- An Adj-SID references a failed or incorrect interface.
- Nested BSIDs create a recursion loop.
- The expanded stack exceeds the ingress platform MSD.
- TI-LFA or entropy labels push the worst-case stack beyond the hardware limit.
- The control plane reports the policy as up, but the hardware entry failed.
- Traffic steering never selects the BSID, so policy counters remain zero.
- Additional labels cause an MTU failure that small probes do not expose.
- Reoptimization leaves a stale LFIB entry or creates a transient blackhole.

## 12. Exit criteria

Declare the BSID implementation ready only when:

- The correct head-end owns a unique BSID.
- The BSID maps to the intended policy and active candidate path.
- Every segment in the expanded list is decoded and reachable.
- Nested policy recursion is loop-free.
- The worst-case label stack stays within platform MSD and MTU limits.
- Control-plane, LFIB and hardware entries agree.
- Test traffic selects the BSID and increments the expected counters.
- Controlled failure testing meets the loss and convergence thresholds.
- Restoration produces the expected candidate-path behavior.
- Evidence and rollback criteria are documented.

## Suggested verification record

Record the date, software release, hardware type, head-end, endpoint, color, BSID, allocation source, active candidate, standby candidate, segment list, expanded label depth, platform MSD, MTU, LFIB action, hardware result, traffic counter, failure-detection time, reoptimization time, loss, restoration result and reviewer sign-off.
