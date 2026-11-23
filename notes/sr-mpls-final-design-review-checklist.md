# SR-MPLS Final Design Review and Production-Readiness Checklist

Use this checklist before approving an SR-MPLS design for production or closing a lab-to-live migration. It brings the complete control plane, forwarding plane, service plane and operations model into one review.

## 1. Underlay and IGP

- Confirm every required OSPF or IS-IS adjacency is stable.
- Verify loopbacks and infrastructure links are advertised with the intended metrics.
- Check ECMP behavior and confirm that asymmetric paths are understood.
- Verify BFD timers and failure-detection coverage.
- Confirm authentication, filtering and passive-interface policy.

## 2. Segment Routing foundation

- Confirm SR-MPLS is enabled on every participating node.
- Verify the SRGB and SRLB ranges are documented and collision-free.
- Confirm Prefix-SIDs are unique and stable.
- Validate Adjacency-SIDs only where explicit link steering is required.
- Check Anycast SIDs for identical function and consistent reachability.
- Verify maximum SID depth and label-stack capability on every platform.

## 3. Forwarding-plane validation

- Compare the IGP database, RIB, CEF/FIB and LFIB.
- Verify each Prefix-SID resolves to the intended next hop.
- Confirm imposed and swapped labels match the design.
- Test MTU with the deepest expected label stack.
- Check PHP, explicit-null and QoS behavior at domain boundaries.
- Validate counters on policies, labels and interfaces.

## 4. Traffic engineering and SR Policies

- Confirm each policy tuple: headend, color and endpoint.
- Inventory local, BGP and PCEP candidate paths.
- Verify candidate-path preference, validity and deterministic tie-breaking.
- Validate metrics, affinities, SRLGs, bandwidth and disjointness constraints.
- Confirm Binding SIDs and segment lists are programmed at the headend.
- Prove that the intended service route is actually steered into the policy.

## 5. Resilience and convergence

- Verify TI-LFA coverage for every protected link and node.
- Test BFD detection, repair activation and post-convergence handoff separately.
- Confirm microloop-avoidance behavior during ordered convergence.
- Measure packet loss and convergence time during controlled failures.
- Test restoration and reversion, not only failure.
- Validate failure behavior when the PCE or controller is unavailable.

## 6. Services over SR-MPLS

- Validate L3VPN route distinguishers, route targets and VPN labels.
- Confirm the outer transport label and inner service label have the expected owners.
- Verify EVPN route types, Ethernet Segment Identifiers and multihoming behavior.
- Test service isolation, ECMP, failover and end-to-end MTU.
- Confirm customer routes do not bypass the intended SR Policy.

## 7. Controller and topology visibility

- Verify PCEP sessions and negotiated stateful capabilities.
- Compare the PCE topology database with the live IGP and BGP-LS view.
- Confirm policy ownership, delegation and fallback behavior.
- Validate topology freshness, constraints and SID resolution before accepting a computed path.
- Test controller restart, session loss and policy resynchronization.

## 8. Operations, security and observability

- Document baseline commands and expected outputs for every layer.
- Export telemetry for adjacency, SID, policy, label, interface and service state.
- Alert on topology changes, policy invalidation and label-programming failures.
- Restrict management and controller access with least privilege and MFA.
- Back up configurations and test restoration.
- Maintain rollback steps, escalation contacts and maintenance-window criteria.

## 9. Acceptance test

Before sign-off, prove all of the following:

1. Unlabelled underlay reachability works.
2. Labelled transport follows the intended Prefix-SID path.
3. SR Policies select and install the correct candidate paths.
4. VPN and EVPN services steer over the expected transport.
5. Failure protection meets the convergence target.
6. Restoration is stable and free of persistent loops.
7. Monitoring detects the injected failures.
8. Rollback returns the network to the approved baseline.

## Final lesson

An SR-MPLS design is production-ready only when the control plane, label forwarding, service steering, resilience and operating model agree. A successful ping proves reachability; production acceptance requires evidence from every layer.
