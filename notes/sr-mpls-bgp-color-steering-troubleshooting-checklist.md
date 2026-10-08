# SR-MPLS BGP Color Steering Troubleshooting Checklist

BGP color steering connects a service route's transport intent to a Segment Routing policy. A route carrying a **BGP Color Extended Community** is matched to an SR policy by the tuple **color + endpoint**. The endpoint is normally derived from the route's BGP next hop.

A policy can be operationally up while no service route is using it. Troubleshooting must therefore prove both sides:

1. The SR policy is valid and programmed.
2. The intended service route resolves through that policy.

> Use platform-specific show commands for your router software and release. Record exact outputs, timestamps, and traffic counters so every conclusion is reproducible.

## 1. Define the expected steering tuple

Before touching the router, write down the intended values.

| Item | Expected value | Why it matters |
|---|---|---|
| Service prefix or VPN route | Prefix/VRF/SAFI | Identifies the consumer route |
| BGP Color Extended Community | Color value | Expresses transport intent |
| BGP next hop | Endpoint address | Must match the policy endpoint |
| SR policy endpoint | IPv4/IPv6 address | Selects the policy destination |
| Candidate path | Preference and origin | Determines the active segment list |
| Binding SID (BSID) | MPLS label or SRv6 SID | Represents the programmed policy |
| Expected egress/path | Nodes and links | Provides a forwarding baseline |

Do not continue with vague values such as “the blue policy.” Capture the numeric color, exact endpoint, address family, route distinguisher, and VRF.

## 2. Capture a baseline

Collect a baseline before changing configuration or failing links:

- Service-route BGP detail, including communities and next hop
- Best-path selection and RIB recursion
- SR policy operational state
- Active candidate path and segment list
- BSID allocation and owner
- MPLS or SRv6 forwarding entries
- Hardware/FIB programming state
- Interface, policy, and tunnel counters
- End-to-end reachability, latency, and loss
- Current topology and IGP reachability to the endpoint

This baseline separates an existing control-plane problem from behavior introduced during a failure test.

## 3. Verify the Color Extended Community

Inspect the exact route received by the steering headend. Confirm that:

- The expected Color Extended Community is present.
- The value is not changed or stripped by import/export policy.
- The community is attached in the correct address family.
- Route reflection preserves the color.
- The selected best path is the path carrying the intended color.
- VPN route import does not select a different uncolored path.
- Add-Path, multipath, or policy ordering does not hide the colored path.

If the color is missing, trace the route hop by hop: confirm the origin attaches it, check outbound policy counters, inspect every route reflector, verify inbound policy at the headend, and compare received, accepted, and best-path attributes.

A correctly configured SR policy cannot steer a route that arrives without the required color.

## 4. Validate endpoint resolution

Color alone is not enough. The route must resolve to the intended policy endpoint.

Verify:

- The BGP next hop equals the SR policy endpoint.
- The endpoint is reachable in the correct routing table.
- Next-hop-self has not changed the next hop unexpectedly.
- Route reflection has not selected a different endpoint.
- The endpoint address family matches the policy.
- VPN or labeled-unicast recursion uses the expected global-table next hop.
- Any route policy derives the endpoint as designed.

A common failure is a valid policy for endpoint A while the service route's next hop is endpoint B. Both objects look healthy when checked independently, but the tuple never matches.

## 5. Confirm policy selection

Inspect the SR policy database for the exact color and endpoint. Confirm that:

- The policy exists for the expected color + endpoint tuple.
- The policy is operationally up.
- The intended candidate path is valid.
- Candidate-path preference selects the expected source.
- A dynamic path has a valid computation result.
- An explicit path has a resolvable ordered segment list.
- Disqualification reasons are absent.
- The policy is installed in the headend forwarding plane.

When multiple candidate paths exist, record their origin, preference, validity, active state, and any failure reason. Do not equate “policy up” with “preferred candidate active.”

## 6. Validate the segment list

For the active candidate path:

- Decode every SID or label.
- Map node SIDs to loopbacks and adjacency SIDs to links.
- Confirm all SIDs are reachable from the imposing node.
- Check that labels are interpreted using the correct SRGB.
- Verify adjacency-SID scope and protection behavior.
- Confirm stack depth is supported by every node and interface.
- Check MTU after label imposition.
- Look for stale topology, PCE, or BGP-LS information.
- Ensure the list reaches the intended endpoint without loops.

If the policy is up but traffic drops, test each segment independently and compare the expected forwarding action with the LFIB.

## 7. Check BSID programming and recursion

The BSID is the bridge between route recursion and the policy segment list. Verify that:

- A BSID is allocated to the active policy.
- The BSID owner is the intended color/endpoint policy.
- The BSID is present in the required RIB/FIB or policy table.
- The BSID resolves to the active segment list.
- The label operation is correct: push, swap, pop, or impose.
- Hardware programming matches the control-plane view.
- No stale BSID from an older policy is present.
- Recursive next-hop depth stays within platform limits.

~~~text
Service route
  -> colored recursive next hop
  -> SR policy (color, endpoint)
  -> active candidate path
  -> BSID
  -> segment list
  -> LFIB/FIB entry
  -> hardware adjacency
~~~

A break anywhere in this chain can leave the policy operational while traffic follows ordinary IGP recursion.

## 8. Prove the service route consumes the policy

Inspect the service route at the steering headend and confirm its resolved next hop points to the SR policy or BSID.

- Route detail shows the expected color and next hop.
- Recursive resolution names the intended policy.
- The installed forwarding entry references the policy or BSID.
- Traffic counters increment on the policy as test traffic is sent.
- Counters on the ordinary IGP path do not increment unexpectedly.
- A controlled flow follows the expected segment list.
- Return-path behavior is tested separately.

Use a destination that unambiguously matches the service route. A successful ping to the endpoint proves endpoint reachability, not that the service prefix is color-steered.

## 9. Compare three layers

| Layer | Evidence | Expected result |
|---|---|---|
| Control plane | BGP route, color, next hop, policy state | Correct tuple and active candidate |
| Forwarding plane | RIB/FIB/LFIB and BSID recursion | Service route resolves through the policy |
| Hardware/data plane | ASIC entry, counters, packet capture | Packets use the expected stack and path |

If control-plane output is correct but counters remain at zero, focus on route installation, VRF context, hardware programming, and the actual test flow.

## 10. Run failure and reoptimization tests

After saving the baseline, fail one component at a time:

1. Fail a link on the preferred segment list.
2. Fail an intermediate node.
3. Withdraw the preferred candidate path.
4. Remove endpoint reachability.
5. Withdraw or change the colored service route.
6. Restore each component and observe reoptimization.

For every test, measure detection time, traffic loss, policy transition, candidate and segment-list changes, BSID stability, FIB/LFIB programming time, counter movement, and restoration behavior.

Confirm that fallback matches the design. Traffic may use a backup candidate, fall back to IGP forwarding, or become unreachable. An unexpected fallback can hide a steering failure.

## 11. Common failure patterns

### Policy is up, but counters stay at zero

- Missing Color Extended Community
- Route next hop does not match the endpoint
- Service route is not the selected best path
- Route uses ordinary IGP recursion
- Wrong VRF or address family
- Hardware entry not programmed

### Correct color, wrong policy

- Endpoint mismatch
- Duplicate color used for multiple endpoints
- Unexpected next-hop-self
- Route reflection changed the selected path
- Policy exists in another routing context

### Correct policy, traffic drops

- Unresolved SID or incorrect SRGB interpretation
- Invalid adjacency SID
- Excessive label-stack depth
- MTU black hole
- Missing LFIB or hardware entry
- Broken endpoint reachability

### Primary failure does not move traffic

- Backup candidate is invalid
- Failure is not visible to the computation source
- Reoptimization timer is longer than expected
- Stale PCE or BGP-LS topology
- Route recursion remains pinned to an old BSID
- Hardware update failed

## 12. Evidence worksheet

| Check | Command/output reference | Expected | Observed | Pass/Fail |
|---|---|---|---|---|
| Service route present | | | | |
| Color attached | | | | |
| BGP next hop correct | | | | |
| Endpoint reachable | | | | |
| Policy tuple matches | | | | |
| Preferred candidate active | | | | |
| Segment list valid | | | | |
| BSID programmed | | | | |
| RIB/FIB/LFIB consistent | | | | |
| Hardware entry present | | | | |
| Service route resolves via policy | | | | |
| Policy counters increase | | | | |
| Failure convergence meets target | | | | |
| Restoration is clean | | | | |

## 13. Exit criteria

Close the incident only when:

- The service route carries the intended color.
- The BGP next hop matches the SR policy endpoint.
- The intended color + endpoint policy is active.
- The expected candidate path and segment list are selected.
- The BSID and recursive forwarding entries are programmed.
- The service route explicitly resolves through the policy.
- Hardware counters prove traffic uses the policy.
- Planned failure tests produce the expected fallback and restoration behavior.
- Evidence is saved with timestamps and the final configuration state.

The key operational lesson is simple: **verify both the SR policy and its consumers**. A healthy policy with no consuming routes is not successful color-based steering.
