# SR-MPLS TI-LFA Failure-Test Evidence Template

Use this worksheet to prove that TI-LFA protects real services during link, node, and SRLG failures. Replace the bracketed fields and attach raw command output where possible.

## 1. Test identification

- Date/time: [YYYY-MM-DD HH:MM TZ]
- Engineer: [name]
- Change or test reference: [ticket]
- Vendor/platform/software: [details]
- Topology version: [diagram or commit]
- Service under test: [L2VPN/L3VPN/SR Policy/Internet]

## 2. Protection objective

- Source/PQ/headend: [node]
- Destination prefix or endpoint: [prefix]
- Protected resource: [link/node/SRLG]
- Required convergence target: [milliseconds]
- Maximum acceptable packet loss: [value]
- Expected repair type: [TI-LFA/remote LFA/local LFA]

## 3. Pre-failure control-plane evidence

Record:

- IGP adjacency state and metric
- Prefix-SID and SRGB/SRLB advertisements
- Primary next hop and outgoing label stack
- Computed repair next hop or PQ node
- Repair segment list
- TI-LFA coverage and reason codes
- BFD or interface failure-detection timers

Example evidence table:

| Check | Expected | Observed | Pass/Fail |
|---|---|---|---|
| Primary next hop | [node/interface] | [result] | [ ] |
| Repair next hop | [node/interface] | [result] | [ ] |
| Repair SID stack | [labels/SIDs] | [result] | [ ] |
| Protected resource excluded | Yes | [result] | [ ] |

## 4. Pre-failure forwarding baseline

- Traffic generator or probe: [tool]
- Flow count and packet rate: [value]
- Packet size and MPLS overhead: [bytes]
- ECMP/LAG path distribution: [result]
- Baseline latency/jitter/loss: [result]
- Service labels and transport labels: [stack]

## 5. Failure execution

Document the exact action:

- Failure type: [link/node/SRLG]
- Injection method: [shutdown/power loss/ACL/test tool]
- Failure timestamp: [time]
- Detection timestamp: [time]
- Repair activation timestamp: [time]
- IGP convergence timestamp: [time]
- Restoration timestamp: [time]

Do not combine multiple failure actions unless the test explicitly targets correlated failure.

## 6. During-failure forwarding evidence

Capture:

- Active repair next hop
- Imposed repair label or SID stack
- Packets lost and loss duration
- Maximum latency and jitter
- Out-of-order or duplicated packets
- Microloop evidence
- ECMP polarization or path imbalance
- CPU, memory, and line-card exceptions
- Service-level alarms

| Measurement | Target | Observed | Pass/Fail |
|---|---:|---:|---|
| Failure detection | [ms] | [ms] | [ ] |
| Repair activation | [ms] | [ms] | [ ] |
| Packet loss | [packets/%] | [result] | [ ] |
| Peak latency | [ms] | [ms] | [ ] |
| Microloop | None | [result] | [ ] |

## 7. Post-convergence and restoration

Verify:

- Traffic moves from repair to the post-convergence path as designed
- No stale repair entries remain
- Label stacks match the intended topology
- All protected services recover
- Restoration does not cause a second unacceptable loss event
- Reoptimization and hold-down timers behave as expected

## 8. Failure variations

Repeat where relevant:

- [ ] Point-to-point link failure
- [ ] Protected-node failure
- [ ] SRLG or shared-conduit failure
- [ ] Parallel-link member failure
- [ ] ECMP next-hop failure
- [ ] BFD session failure
- [ ] Reverse-direction failure
- [ ] Failure during an active topology change

## 9. Exceptions and root cause

For each failed check, record:

1. Symptom and timestamp
2. Expected behavior
3. Control-plane state
4. Data-plane state
5. Platform limitation or defect reference
6. Corrective action
7. Retest result

## 10. Exit criteria

The test is complete only when:

- [ ] The protected resource is excluded from the repair path
- [ ] Measured loss and convergence meet the service objective
- [ ] No sustained loop, black hole, or unintended ECMP path occurs
- [ ] All required service types have been validated
- [ ] Restoration behavior is acceptable
- [ ] Raw evidence and topology references are stored
- [ ] Exceptions have owners and retest dates

## Suggested evidence naming

`evidence/<date>/<source>-to-<destination>/<failure-type>/`

Keep timestamps synchronized across routers, probes, and traffic generators so control-plane events can be correlated with packet loss.
