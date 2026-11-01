# SR-MPLS Anycast SID Troubleshooting Checklist

An Anycast SID allows multiple routers to advertise the same Segment Routing prefix-SID. The IGP then directs traffic toward the nearest reachable advertising node, which can provide ECMP, service locality, and resilient failover.

When traffic reaches the wrong node, follows an unexpected path, or fails completely, validate the control plane before changing labels or policies.

## 1. Confirm the Anycast definition

- Verify that every intended node advertises the same Anycast prefix.
- Confirm that the same prefix-SID or SID index is configured on each member.
- Check that the prefix is treated as an Anycast service prefix rather than a unique loopback.
- Make sure no unintended router advertises the same prefix or SID.

## 2. Validate SRGB label resolution

If the Anycast SID is advertised as an index, each receiving router calculates the local label as:

```text
local label = local SRGB base + SID index
```

The numeric label can therefore differ between routers when SRGB bases differ. What must remain consistent is the SID index and its meaning.

Check:

- SRGB range on every transit and Anycast node.
- SID index falls inside each relevant SRGB.
- No label collision exists with another prefix-SID.
- The expected incoming label is installed in the LFIB.

## 3. Verify the IGP view

- Confirm the Anycast prefix is present in IS-IS or OSPF from every intended member.
- Compare IGP metrics from the headend to each advertising node.
- Verify whether ECMP is expected and actually installed.
- Check for filtering, summarization, route leaking, or area/level boundaries that hide an advertisement.
- Confirm the topology and metric database are synchronized.

The headend cannot choose the correct Anycast instance if its IGP view is incomplete or inconsistent.

## 4. Inspect RIB, CEF/FIB, and LFIB

Validate the forwarding chain in order:

1. The Anycast prefix exists in the RIB.
2. The expected IGP next hop or ECMP set exists in CEF/FIB.
3. The prefix-SID resolves to the expected local label.
4. The LFIB contains the correct swap, pop, or forward action.
5. Interface and label counters increase while test traffic runs.

If the RIB is correct but the FIB or LFIB is not, investigate forwarding-plane programming rather than changing the IGP.

## 5. Test failure behavior

- Remove one Anycast member from service in a controlled test.
- Measure detection and convergence time.
- Confirm traffic moves to another advertising member.
- Verify that BFD, TI-LFA, or another protection mechanism behaves as designed.
- Repeat the test for both the primary and ECMP cases.

## Quick decision tree

```text
Anycast prefix missing?
  -> Check IGP advertisement, filtering, and area/level reachability.

Prefix present but wrong node selected?
  -> Compare IGP metrics and ECMP next hops.

RIB correct but label path missing?
  -> Check SRGB calculation, FIB, and LFIB programming.

Traffic works until a member fails?
  -> Validate failure detection, repair path, and convergence.
```

## Key takeaway

An Anycast SID is only as deterministic as the IGP view and forwarding state behind it. Start with reachability and metrics, verify SID-to-label resolution, and then prove the behavior with counters and failure tests.

