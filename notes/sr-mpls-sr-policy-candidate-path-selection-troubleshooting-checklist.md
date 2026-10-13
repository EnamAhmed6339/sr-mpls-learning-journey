# SR Policy Candidate-Path Selection Troubleshooting Checklist

Use this checklist when an SR Policy exists but the headend selects the wrong path, does not install a candidate path, or fails to move to the expected backup.

## 1. Define the expected policy

Record the intended policy before examining the implementation.

- [ ] Headend:
- [ ] Color:
- [ ] Endpoint:
- [ ] Address family:
- [ ] Expected candidate-path origin:
- [ ] Expected preference:
- [ ] Expected segment list:
- [ ] Expected Binding SID (BSID):
- [ ] Consuming service or route:
- [ ] Expected protection and convergence target:

The policy key is the color and endpoint tuple. Confirm that every control-plane and service reference uses the same values.

## 2. Inventory every candidate path

List all candidates for the policy, including paths that are configured but currently invalid.

| Candidate | Origin | Protocol/source | Preference | Valid | Active | Disqualification reason |
|---|---|---|---:|---|---|---|
| Primary | Local / BGP / PCEP |  |  |  |  |  |
| Backup 1 | Local / BGP / PCEP |  |  |  |  |  |
| Backup 2 | Local / BGP / PCEP |  |  |  |  |  |

- [ ] Confirm that no unexpected local, BGP SR Policy, or PCEP candidate exists.
- [ ] Verify that the controller or BGP peer advertises the expected candidate.
- [ ] Check whether stale state remains after a controller, session, or configuration change.
- [ ] Confirm that policy identifiers and discriminator values do not create a different candidate than intended.

## 3. Verify selection logic

The headend normally selects the highest-preference candidate path that is valid.

- [ ] Compare preference values numerically.
- [ ] Confirm which candidate is selected and why.
- [ ] Check whether the preferred candidate is invalid, incomplete, or administratively disabled.
- [ ] Record the explicit invalid or disqualification reason.
- [ ] Verify origin-specific rules and tie breakers for the platform.
- [ ] Confirm that a newly learned candidate did not replace the intended candidate.
- [ ] Check whether a withdrawn candidate remains visible only as historical or stale state.

A configured backup is not protection unless it is valid and eligible for selection.

## 4. Validate the active segment list

For the selected candidate:

- [ ] Confirm the intended segment list is active.
- [ ] Validate the order of every SID or MPLS label.
- [ ] Resolve each Prefix-SID, Adj-SID, Node-SID, or BSID to its owner.
- [ ] Confirm the SID is advertised and reachable in the correct topology or algorithm.
- [ ] Verify SRGB and SRLB interpretation on every relevant node.
- [ ] Check adjacency state for strict-hop Adj-SIDs.
- [ ] Confirm MSD limits are not exceeded.
- [ ] Verify that the segment list does not create a loop or unreachable hop.
- [ ] Check that the selected list satisfies metric, affinity, bandwidth, and exclusion constraints.

## 5. Check BSID programming

- [ ] Confirm that the policy has the expected BSID.
- [ ] Verify that the BSID expands to the active segment list.
- [ ] Check the RIB, FIB, LFIB, and hardware forwarding state.
- [ ] Confirm the outgoing interface and next-hop resolution.
- [ ] Verify that local-label allocation does not conflict with another function.
- [ ] Confirm that a candidate change updates the BSID programming.
- [ ] Check for control-plane and forwarding-plane state mismatch.

## 6. Verify route and service consumption

A healthy SR Policy has no effect unless traffic is steered into it.

- [ ] Identify the exact BGP route, VRF, tunnel, service, or flow that should consume the policy.
- [ ] Confirm that the route carries the intended Color Extended Community.
- [ ] Verify that the route next hop matches the policy endpoint.
- [ ] Confirm that the route resolves recursively through the SR Policy.
- [ ] Check the installed forwarding entry for the service prefix.
- [ ] Verify that a default or lower-priority route is not winning instead.
- [ ] Test the actual traffic class and source/destination pair.
- [ ] Confirm counters increase on the expected policy and segment list.

## 7. Test failover and reoptimization

- [ ] Capture the active candidate, segment list, BSID, and counters before the test.
- [ ] Fail the active candidate using an approved maintenance method.
- [ ] Confirm that the primary candidate becomes invalid or is withdrawn.
- [ ] Verify that the next-highest-preference valid candidate becomes active.
- [ ] Measure packet loss and convergence time.
- [ ] Confirm that the BSID now expands to the backup segment list.
- [ ] Verify service-route resolution and traffic counters after failover.
- [ ] Restore the primary path.
- [ ] Confirm the intended revertive or non-revertive behavior.
- [ ] Check for oscillation, repeated reoptimization, or stale forwarding state.

## 8. Evidence matrix

Collect evidence at the same timestamp when possible.

| Layer | Evidence |
|---|---|
| Policy key | Color, endpoint, address family |
| Candidate inventory | Origin, preference, validity, active state |
| Disqualification | Exact reason and timestamp |
| Segment list | SID order and resolution |
| BSID | RIB/FIB/LFIB/hardware programming |
| Consumer | Route color, next hop, recursive resolution |
| Traffic | Interface, policy, label, and drop counters |
| Failure test | Event time, convergence, selected backup |
| Recovery | Restored state and reoptimization result |

## 9. Common failure patterns

- Color or endpoint mismatch between the route and policy.
- Higher-preference candidate exists but is invalid.
- Unexpected local or controller-originated candidate wins.
- PCEP or BGP session is up while the candidate itself is stale.
- Segment list contains an unreachable SID or failed adjacency.
- SRGB/SRLB interpretation produces an unexpected local label.
- MSD or policy constraints invalidate the preferred path.
- BSID points to an old segment list after reoptimization.
- Route carries no color or resolves through a different next hop.
- Control plane shows the policy while hardware programming is missing.
- Backup candidate is configured but invalid before the failure occurs.

## 10. Exit criteria

Close the incident only when all of the following are true:

- [ ] The intended highest-preference valid candidate is active.
- [ ] The active segment list matches the design.
- [ ] Every SID resolves and is programmed correctly.
- [ ] The BSID expands to the current active list.
- [ ] The consuming route resolves through the SR Policy.
- [ ] Production traffic follows the intended path.
- [ ] Backup selection and convergence have been tested.
- [ ] Evidence, timestamps, and corrective actions are documented.

## Quick decision flow

1. Confirm the color and endpoint tuple.
2. Inventory every candidate and its origin.
3. Select the highest-preference valid candidate.
4. Explain why any preferred candidate is invalid.
5. Validate the active segment list and every SID.
6. Verify BSID and hardware programming.
7. Prove that the service route consumes the policy.
8. Test failure, backup selection, and recovery.
