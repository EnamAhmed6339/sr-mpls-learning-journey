# L3VPN Over SR-MPLS

A hands-on lab proving MPLS L3VPN (RFC 4364) running entirely on a Segment
Routing data plane instead of LDP — same VRF/RD/RT/VPNv4 control plane,
SR-MPLS transport underneath.

![Network diagram](diagrams/L3VPN_Segment_Routing_Network_Diagram.png)

## Topology

Four Cisco IOS XRv routers, OSPF area 0 with `segment-routing mpls` enabled,
single SRGB block `16000-23999` shared by all nodes.

| Router | Role | Loopback0 (Node-SID) | Customer-facing VRFs |
| --- | --- | --- | --- |
| R1 | PE | 1.1.1.1 (index 1) | Accounting (192.168.15.0/24), HR (192.168.16.0/24) |
| R2 | P | 2.2.2.2 (index 2) | — |
| R3 | P | 3.3.3.3 (index 3) | — |
| R4 | PE | 4.4.4.4 (index 4) | Accounting (192.168.47.0/24), HR (192.168.48.0/24) |

Core links: R1–R2 (`10.1.12.0/24`), R2–R3 (`10.1.23.0/24`), R3–R4 (`10.1.34.0/24`).

**BGP:** single AS 12345, `address-family vpnv4 unicast` between the two PEs
(R1 and R4), core-facing IGP is OSPF with SR-MPLS as the only label
distribution mechanism — no LDP anywhere in the core.

**VRFs:**

| VRF | Route Target | R1 subnet | R4 subnet |
| --- | --- | --- | --- |
| Accounting | 1:100 | 192.168.15.0/24 | 192.168.47.0/24 |
| HR | 1:200 | 192.168.16.0/24 | 192.168.48.0/24 |

## What this proves

- **VRF-to-VRF reachability across the core** — CEs in the same VRF on
  opposite PEs reach each other; CEs in different VRFs stay isolated, with
  route leaking controlled purely by route-target import/export.
- **SR-MPLS carries the transport label, BGP carries the VPN label** — a
  packet from R5 (behind R1, VRF Accounting) to R7 (behind R4, VRF
  Accounting) is forwarded with two MPLS labels: the SR transport label
  (`16004`, R4's Prefix-SID, swapped hop-by-hop) and the VPN label
  (`24002`, learned via VPNv4 BGP, unchanged end-to-end) — confirmed with
  `traceroute` in [`verification_and_remarks.txt`](verification_and_remarks.txt).
- **Per-VRF aggregate labels on the PE** — `show mpls forwarding` on R1 shows
  the two VRFs each getting their own aggregate label (`24001`/`24002`) for
  traffic destined to the local CEs.
- **VPNv4 exchange between PEs** — `show bgp vpnv4 unicast` on R1 shows both
  VRFs' routes with their Route Distinguishers (`1:1` Accounting, `2:2` HR)
  and the remote PE (`4.4.4.4`) as next hop.

## Repository contents

| Path | Contents |
| --- | --- |
| [`configs/`](configs) | Full `show running-config` for all four routers (`xrv1`–`xrv4`) |
| [`verification_and_remarks.txt`](verification_and_remarks.txt) | Ping, traceroute, `show mpls forwarding`, `show bgp vpnv4 unicast`, `show bgp vrf` output with inline commentary |
| [`diagrams/`](diagrams) | Network topology diagram and final lab output screenshot |
| [`docs/`](docs) | [Advanced technical paper](docs/SR_MPLS_L3VPN_Advanced_Technical_Paper.docx) (.docx), [lab write-up](docs/SR_MPLS_L3VPN.pdf) (.pdf), and [workbook](docs/Workbook_SR-MPLS_L3VPN.pdf) (.pdf) |
| [`banners/`](banners) | Section banners summarizing topology, addressing, protocol architecture, deployment workflow, troubleshooting, and interview-prep talking points |

## Key takeaway

L3VPN's control plane (VRF, RD, RT, VPNv4 BGP) is completely decoupled from
the transport label distribution mechanism. Swapping LDP for SR-MPLS changes
only how the transport label is signaled — the VPN service, the PE
configuration model, and the forwarding behavior for customer traffic stay
the same.
