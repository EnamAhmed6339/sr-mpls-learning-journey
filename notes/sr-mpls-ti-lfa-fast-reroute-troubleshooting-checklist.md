# SR-MPLS TI-LFA Fast-Reroute Troubleshooting Checklist

## Purpose

Topology-Independent Loop-Free Alternate (TI-LFA) protects SR-MPLS traffic by precomputing a post-convergence repair path before a failure occurs. When a protected link, node, or shared-risk link group (SRLG) fails, the point of local repair (PLR) imposes a repair segment list and forwards traffic while the IGP converges.

This checklist provides a vendor-neutral workflow for proving that TI-LFA protection is calculated, installed, activated, and removed correctly.

## Core concepts

- **PLR:** The router directly adjacent to the protected resource that activates the repair.
- **Post-convergence path:** The path the IGP would select after the failed resource is removed from the topology.
- **P node:** A node reachable from the PLR without traversing the protected resource.
- **Q node:** A node that can reach the destination without traversing the protected resource.
- **PQ node:** A node satisfying both P-space and Q-space conditions.
- **Repair segment list:** Prefix-SIDs and, when required, Adj-SIDs that steer traffic from the PLR to the post-convergence path.
- **Protection scope:** Link, node, or SRLG protection. A valid link-protecting repair may not provide node or SRLG protection.

## 1. Capture the pre-change baseline

Record evidence before enabling, changing, or testing TI-LFA:

- IGP neighbor and adjacency state
- Link metrics, levels or areas, and topology membership
- Primary route and primary next hop to every test destination
- SRGB and SRLB ranges on every router in the repair domain
- Prefix-SID and Adj-SID advertisements
- MPLS forwarding entries and imposed label stacks
- ECMP members, interface counters, and hardware drop counters
- Current convergence time, loss, latency, and jitter
- Platform software release, line-card type, and label-stack limits

## 2. Validate topology and IGP readiness

1. Confirm that the protected interface and destination participate in the intended IGP level, area, topology, and address family.
2. Verify stable adjacencies and bidirectional reachability.
3. Confirm wide-metric or extended-TLV support where required.
4. Check that the topology contains a real alternate path after the protected link or node is removed.
5. Review overload, stub, affinity, admin-group, flex-algorithm, and SRLG constraints that could invalidate the alternate.
6. Confirm the primary path is the expected shortest path before testing a repair.

A topology with no constraint-compliant post-convergence path cannot produce a TI-LFA repair.

## 3. Confirm Segment Routing readiness

- Verify Segment Routing is enabled for the correct IGP instance and address family.
- Confirm every required loopback or node prefix advertises the intended Prefix-SID.
- Verify Adj-SIDs for links that may appear in a repair segment list.
- Check that SRGB ranges do not overlap reserved or locally used labels.
- Confirm that each router calculates the correct local label for every advertised SID index.
- Validate SRLB allocation and Adj-SID programming on the PLR and downstream nodes.
- Confirm the MPLS data plane is enabled on every relevant interface.

## 4. Verify protection intent

Document the intended failure model for each protected resource:

| Protected resource | Required scope | Expected PLR | Destination set | Expected repair |
|---|---|---|---|---|
| Core link | Link protection | Adjacent upstream router | Critical loopbacks and services | Avoid failed link |
| Transit node | Node protection | Upstream neighbor | Critical loopbacks and services | Avoid failed node and its links |
| Shared conduit | SRLG protection | Affected upstream router | Critical destinations | Avoid every member of the SRLG |

Then confirm that the router reports the same protection type. Do not treat a link-protecting backup as proof of node or SRLG protection.

## 5. Confirm the backup next hop is installed

For each protected destination:

1. Display the primary next hop.
2. Display the TI-LFA backup next hop.
3. Confirm the backup is programmed before any failure.
4. Verify the backup does not use the protected link, node, or SRLG.
5. Check whether ECMP changes the PLR or the protection requirement.
6. Confirm the route, LFIB, and hardware forwarding table agree.

If no backup is installed before the event, shortening IGP timers will not create fast reroute protection.

## 6. Inspect the repair segment list

Decode every label in the imposed repair stack:

- Map Prefix-SIDs to their owning nodes or prefixes.
- Map Adj-SIDs to the exact local outgoing adjacency.
- Confirm the first segment is reachable without using the protected resource.
- Confirm the remaining segments steer traffic onto the post-convergence path.
- Verify that no repair segment resolves recursively through the failed resource.
- Check penultimate-hop popping, explicit-null, and service-label behavior.
- For VPN traffic, distinguish transport repair labels from the VPN or service label.

A repair stack can look syntactically valid while still resolving through the resource it is meant to avoid. Always validate recursion and forwarding behavior hop by hop.

## 7. Validate SRGB, SRLB, and label-stack programming

- Recalculate each Prefix-SID label from the local SRGB base and SID index.
- Verify incoming and outgoing labels when routers use different SRGBs.
- Confirm Adj-SIDs use labels from the correct local allocation range.
- Compare control-plane label calculations with the LFIB and hardware table.
- Check maximum SID depth and maximum imposed label-stack depth.
- Include transport, repair, binding, entropy, and service labels in the depth calculation.
- Validate MTU after adding the repair labels; every MPLS label adds four bytes.

## 8. Run a controlled failure test

Use an approved maintenance window and a reversible failure method.

### Before the failure

- Start continuous probes with timestamps and fixed packet sizes.
- Capture interface, route, LFIB, and drop counters.
- Confirm the backup is installed and eligible.
- Start packet capture or telemetry if available.

### During the failure

1. Fail only the intended link or node.
2. Record failure-detection time.
3. Confirm the PLR activates the TI-LFA backup immediately.
4. Verify the repair label stack on transmitted packets.
5. Measure packet loss, latency spike, and microloop duration.
6. Confirm traffic avoids the protected resource.
7. Compare software and hardware counters.

### After IGP convergence

- Confirm the normal post-convergence path replaces the temporary repair.
- Verify the repair labels are removed when no longer required.
- Check all protected services, not only an infrastructure ping.

### After restoration

- Confirm the original primary path returns according to policy.
- Verify that the backup is recalculated and reinstalled.
- Check for oscillation, stale LFIB entries, or repeated microloops.

## 9. Evidence matrix

| Layer | Evidence to collect | Healthy result | Failure indicator |
|---|---|---|---|
| Topology | IGP database and metrics | Alternate path exists | No valid PQ or post-convergence path |
| Protection | TI-LFA detail per prefix | Correct link/node/SRLG scope | Wrong scope or no backup |
| Control plane | Prefix-SID and Adj-SID advertisements | SIDs reachable and unique | Missing, duplicate, or unreachable SID |
| Label allocation | SRGB/SRLB tables | Labels map to intended SIDs | Range mismatch or collision |
| Forwarding | RIB, LFIB, and hardware entries | Primary and backup programmed | Control/data-plane disagreement |
| Repair stack | Imposed labels at PLR | Stack avoids protected resource | Recursive path crosses failure |
| Performance | Loss, latency, convergence | Within rollback threshold | Excess loss or slow activation |
| Platform | MSD, MTU, and resource use | Within supported limits | Stack too deep, MTU drop, resource exhaustion |

## 10. Common failure patterns

- TI-LFA is enabled in the wrong IGP level, area, topology, or address family.
- The topology has no valid post-convergence path.
- Only link protection is available when node protection is required.
- SRLG membership is missing or inconsistent.
- A Prefix-SID or Adj-SID required by the repair is absent or unreachable.
- SRGB/SRLB inconsistencies cause the expected label to map incorrectly.
- The control plane calculates a repair, but the hardware table does not install it.
- The repair stack exceeds the platform's maximum SID depth.
- Extra labels cause an MTU failure that small probes do not reveal.
- Bidirectional Forwarding Detection or physical failure detection is slower than expected.
- A service or VPN label is mistaken for a repair label during packet analysis.
- The backup activates, but the post-convergence transition creates a microloop.

## 11. Exit criteria

Declare TI-LFA ready only when all of the following are true:

- The required protection scope is explicitly verified.
- A valid backup next hop is installed before failure.
- The repair segment list is decoded and avoids the protected resource.
- SRGB, SRLB, LFIB, and hardware tables agree.
- Label depth and MTU remain within platform limits.
- Controlled failure testing meets the loss and convergence targets.
- Traffic transitions cleanly to the post-convergence path.
- The backup is recalculated after restoration.
- Evidence and rollback thresholds are documented.

## Suggested verification record

Record the date, topology version, software release, protected resource, PLR, destination, primary path, backup next hop, repair labels, detection time, repair activation time, total loss, post-convergence path, restoration result, and reviewer sign-off for every test case.
