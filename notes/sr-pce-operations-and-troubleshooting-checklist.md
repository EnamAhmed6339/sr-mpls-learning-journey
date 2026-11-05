# SR-PCE Operations and Troubleshooting Checklist

Segment Routing removes per-flow state from the core, but an SR Policy computed by a Path Computation Element (PCE) still depends on accurate topology, synchronized state, valid constraints, and successful programming at the headend.

Use this checklist when a controller-computed path is missing, invalid, or different from the intended path.

## 1. Verify PCEP Session Health

- Confirm the PCC-to-PCE session is established and stable.
- Check source addresses, TCP reachability, authentication, keepalive and dead-timer values.
- Review session resets, capability mismatches and repeated reconnects.
- Confirm both sides negotiated the required stateful and Segment Routing capabilities.

## 2. Validate the PCE Topology Database

- Confirm the PCE receives the required IGP topology through BGP-LS or another supported source.
- Compare nodes, links, metrics and Prefix-SIDs against the live IGP database.
- Verify TE attributes, affinities, SRLG membership and available bandwidth.
- Check for stale links or delayed withdrawals after a failure.

## 3. Confirm Policy Identity and Ownership

- Match the policy headend, endpoint and color.
- Identify whether the policy is delegated to the PCE or controlled locally.
- Confirm the expected candidate-path origin and preference.
- Check that a local or BGP candidate is not winning over the PCE candidate.

## 4. Validate Constraints and Objective

- Confirm the requested objective: IGP metric, TE metric, delay or another optimization.
- Validate affinity include/exclude rules.
- Check SRLG, node and link disjointness requirements.
- Confirm bandwidth, maximum metric and maximum SID-depth constraints are achievable.
- Treat "no path" as a constraint problem until the topology proves otherwise.

## 5. Inspect the Computed Segment List

- Resolve every Prefix-SID, Adjacency-SID and Binding-SID.
- Confirm strict hops are still adjacent and loose hops remain reachable.
- Check label-stack depth against platform MSD.
- Verify the list does not depend on a failed, filtered or unadvertised SID.

## 6. Verify Headend Programming

- Confirm the PCE candidate is valid and selected.
- Check the Binding-SID and active segment list in the forwarding plane.
- Verify the intended route or service resolves through the policy.
- Compare policy counters with interface, MPLS and service counters.

## 7. Test Failure and Re-optimization

- Fail a protected link or node in a controlled window.
- Measure local protection, PCE recomputation and policy update timing separately.
- Confirm the new path meets the original constraints.
- Verify traffic returns safely after restoration and does not oscillate between paths.

## Fast Diagnostic Order

1. PCEP session
2. PCE topology freshness
3. Policy tuple and ownership
4. Candidate-path preference and validity
5. Constraint feasibility
6. Segment-list resolution
7. Headend forwarding state
8. Failure and restoration behavior

## Key Lesson

A PCE can compute only from the topology and attributes it knows. When the selected path looks wrong, validate topology freshness and policy ownership before changing the constraint set.

