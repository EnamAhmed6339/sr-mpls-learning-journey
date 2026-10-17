# SR-MPLS Three-Plane Troubleshooting Checklist

Use this sequence to separate route computation, label programming, and service failures before changing configuration.

## 1. Define the symptom

- Record the affected source, destination, and service.
- Note the first failure timestamp and any recent topology, policy, or software changes.
- Define the scope: one flow, one VPN, one node, or network-wide.
- Compare the expected path with the observed path.

## 2. Check the control plane

- Verify IGP adjacencies, metrics, and topology.
- Confirm Prefix-SID and Adjacency-SID advertisements.
- Check SRGB and SRLB consistency.
- Verify BGP VPN routes and SR Policy candidate-path validity.
- Confirm the topology database and RIB agree.

## 3. Check the forwarding plane

- Compare the RIB with CEF/FIB programming.
- Verify LFIB incoming and outgoing labels.
- Confirm the imposed segment list and next hop.
- Check ECMP or LAG member selection.
- Verify MTU after label imposition.
- Review adjacency resolution, interface errors, and drops.

## 4. Check the service plane

- Verify the VRF route and route target.
- Confirm the VPN or service label.
- Check L2VPN pseudowire or EVPN state where applicable.
- Test endpoint reachability.
- Rule out ACL, QoS, firewall, and uRPF effects.

## 5. Correlate the evidence

| Observation | Control plane | Forwarding plane | Likely direction |
|---|---|---|---|
| Route absent | Absent | Absent | IGP, BGP, or policy |
| Route present but label path missing | Present | LFIB missing or wrong | Programming or platform |
| Route and labels correct but traffic drops | Present | Present | MTU, interface, ACL, or service |
| Only one ECMP path fails | Present | One member abnormal | Adjacency or link |

## 6. Capture evidence

- Timestamps and affected flows
- Relevant show-command output before and after changes
- Packet loss, latency, and jitter
- Label stack at each hop
- Packet captures or traffic-generator results
- Alarms and logs
- Platform and software versions

## 7. Recover safely

- Fix the smallest proven fault.
- Avoid simultaneous configuration changes.
- Record the baseline before intervention.
- Verify failover and restoration behavior.
- Preserve evidence for the root-cause review.

## Exit criteria

- [ ] Route, label, forwarding, and service state are restored.
- [ ] No sustained loss, loop, or unexpected path remains.
- [ ] Evidence is stored with timestamps.
- [ ] A root-cause owner and retest date are assigned.

---

Prepared as part of Enam Ahmed's SR-MPLS learning journey on 17 October 2026.