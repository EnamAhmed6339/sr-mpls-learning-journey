# SR Policy Active but No Traffic: Validation Checklist

An SR Policy can be operational on the headend while the intended service traffic still follows a different path. Use this checklist to trace the design from intent to forwarding before changing the underlay.

## 1. Confirm the policy key

- Verify the policy headend.
- Confirm the color matches the intended transport objective.
- Confirm the endpoint resolves to the expected destination address.
- Check that the policy is operational and not only administratively enabled.

## 2. Validate candidate-path selection

- List every candidate path for the policy.
- Confirm the expected path has the highest valid preference.
- Record the reason for any inactive or invalid candidate path.
- Verify whether the active path is explicit, dynamic, or delegated.

## 3. Inspect the active segment list

- Confirm every Prefix-SID and Adjacency-SID exists on the intended node or link.
- Validate SID order against the required path.
- Check that the label stack fits the platform depth and path MTU limits.
- Confirm recursive next-hop and label resolution in the forwarding plane.

## 4. Verify the Binding SID

- Confirm the Binding SID is allocated on the correct headend.
- Verify it resolves to the selected candidate path and active segment list.
- Check for collisions, stale programming, or unsupported recursive behavior.

## 5. Prove service-to-policy steering

- Verify the service route carries the intended color or steering attribute.
- Confirm the route next hop matches the SR Policy endpoint.
- Check that the route, VRF, tunnel, or service is actually associated with the policy.
- Compare RIB state with CEF, LFIB, or equivalent forwarding entries.

## 6. Test real forwarding

- Generate traffic that matches the intended service.
- Confirm policy and interface counters increase on the expected path.
- Trace the imposed label stack and verify each hop.
- Test the planned failure and confirm the expected repair or backup behavior.
- Recheck the path after IGP and policy convergence complete.

## Evidence to capture

- Policy color, endpoint, state, and Binding SID
- Candidate paths, preferences, and inactive reasons
- Active segment list
- Service route and steering attributes
- RIB, CEF, and LFIB entries
- Interface and policy counters before and after the test
- Failure-detection and convergence timestamps

## Key lesson

An active SR Policy proves that the policy exists and has a usable path. It does not prove that the intended traffic is being steered into it. Successful validation must connect service intent, policy selection, forwarding programming, and observed traffic behavior.
