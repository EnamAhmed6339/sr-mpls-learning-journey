# BFD and TI-LFA Validation Checklist

Use this checklist to validate fast failure detection, local repair, and final convergence in an SR-MPLS network.

## 1. Define the protection objective

- Record the protected service, source, destination, and traffic direction.
- Define the maximum acceptable packet loss and recovery time.
- Identify whether link, node, or shared-risk protection is required.
- Document the expected primary, repair, and final converged paths.

## 2. Validate BFD detection

- Confirm the BFD session is Up in both directions.
- Record local and remote discriminators.
- Verify negotiated transmit interval, receive interval, and detection multiplier.
- Confirm whether the session is hardware-offloaded or software-based.
- Check platform scale limits and current session count.
- Correlate any flap with interface errors, optics, drops, CPU, or control-plane events.
- Verify that BFD is bound to the intended interface, adjacency, or path.

## 3. Validate TI-LFA protection

- Confirm TI-LFA is enabled for the intended topology and address family.
- Verify that the protected prefix has repair coverage.
- Identify the repair next hop and protection type.
- Inspect the imposed repair label stack.
- Confirm the repair avoids the protected link or node as designed.
- Check label-stack depth and MTU impact.
- Validate ECMP behavior when multiple repair choices exist.

## 4. Test the failure sequence

1. Capture the baseline route, next hop, LFIB, and traffic statistics.
2. Start continuous traffic in both directions.
3. Trigger one controlled failure at a time.
4. Measure packet loss, recovery time, latency, and jitter.
5. Confirm traffic moves to the TI-LFA repair path.
6. Verify the IGP converges to the final path.
7. Restore the failure and validate revertive behavior.

## 5. Correlate the layers

| Signal | Expected observation | If missing |
|---|---|---|
| BFD session Down | Failure detected within the negotiated interval | Check session binding, timers, offload, and platform load |
| Repair next hop active | Traffic immediately uses the local repair | Check TI-LFA coverage and FIB/LFIB programming |
| IGP convergence complete | Final shortest path replaces the repair | Check adjacency, SPF, and route installation |
| Service restored | End-to-end traffic meets the loss target | Check VRF, service labels, MTU, ACL, QoS, and endpoint state |

## 6. Evidence to save

- BFD session details before and after the event
- IGP topology and route changes
- CEF/FIB and LFIB output
- Primary, repair, and final label stacks
- Interface counters and optical levels
- Packet-loss and recovery measurements
- Timestamped logs and alarms
- Platform and software versions

## Exit criteria

- [ ] BFD reliably detects the intended failure.
- [ ] TI-LFA installs and uses the expected repair path.
- [ ] The final converged path is correct.
- [ ] Loss and recovery meet the service objective.
- [ ] No loop, black hole, MTU issue, or unexpected ECMP behavior remains.
- [ ] Evidence, owner, and retest date are recorded.

---

Prepared as part of Enam Ahmed's SR-MPLS learning journey on 25 October 2026.