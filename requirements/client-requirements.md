# Client Requirements Specification
**Project:** CMPG325-2026-147 | **Client ID:** CLI-147
**Client:** Kopano Fibre & Wireless ISP (Mahikeng) | **Industry:** Telecommunications
**Student:** Wana, Kamo (52060586)

## 1. Client Context

Kopano Fibre & Wireless ISP is the client scenario assigned by the CMPG 325 project brief
(CMPG325-2026-147). I searched for public information on this organisation and found no
credible results, so I'm treating Kopano as the assigned academic client scenario, and the
project brief itself as the authoritative source of every requirement below. I haven't
assumed anything about Kopano's actual operations, staff, or infrastructure beyond what the
brief states.

## 2. Requirement Classification

Every design decision in this project traces to one of three sources:

| Tag | Meaning |
|---|---|
| 🟢 Brief | Explicitly stated in the CMPG325-2026-147 project brief |
| 🟡 Industry-supported | Reasonable inference, backed by how comparable telecom/ISP offices are commonly structured |
| 🟠 Design assumption | A decision made by the student to satisfy a brief requirement, explicitly labelled as such |

## 3. Requirements Traceability Matrix

| # | Requirement | Source | Network Implication | Design Response |
|---|---|---|---|---|
| 1 | Multi-VLAN network with controlled inter-VLAN communication | 🟢 Brief §7, §9 | Need at least 3 logical broadcast domains with policy-based routing between them | 4 VLANs: Admin, Technical/NOC, Shared Printers, Contractor Wireless |
| 2 | "Cross departments" implies at least two departments | 🟢 Brief §8 (implied) | Need a minimum of two distinct user groups | 🟠 Admin/Business and Technical/NOC, the minimal split sufficient for a small telecom office, chosen because it's the smallest set that makes the constraint meaningful |
| 3 | Printer sharing must cross departments; file sharing must not | 🟢 Brief §8 | Need one resource reachable by both departments and one resource isolated per department | 🟠 Dedicated Printer VLAN reachable from both Admin and Technical; direct Admin-to-Technical traffic denied by an extended ACL |
| 4 | DHCP: advanced scoped multi-VLAN address assignment | 🟢 Brief §9 | Each VLAN subnet requires its own DHCP scope, matching default gateway, and correct pool boundaries | 4 DHCP scopes on one server, one per VLAN, each scoped to its VLAN's subnet |
| 5 | DHCP must operate across VLAN boundaries via relay/routing | 🟢 Brief §9 | A client's DHCPDISCOVER is the first message in the DORA exchange (Discover, Offer, Request, Acknowledge), and it's a Layer 2 broadcast, so it's confined to its own VLAN's broadcast domain and never reaches a DHCP server sitting in a different VLAN | 🟠 DHCP server placed in VLAN 10; `ip helper-address` configured on R1's VLAN 20, 30, and 40 sub-interfaces, converting each VLAN's broadcast to a unicast and stamping the sub-interface's own address into the packet's `giaddr` field so the server can identify which VLAN the request came from and answer with the matching scope |
| 6 | Exclusions/reserved addresses for infrastructure | 🟢 Brief §9 | Gateways, the DHCP server itself, and any statically-addressed devices must not be leased out dynamically | Gateway addresses excluded from every pool; the DHCP server's own address excluded from VLAN 10's pool; the printer's address held as a MAC-based reservation rather than a plain exclusion, so it stays DHCP-managed with a predictable address |
| 7 | At least 3 VLANs including a dedicated limited-access wireless segment for contractors | 🟢 Brief §9 | Need a physically/logically separate wireless segment | Dedicated Contractor VLAN, wireless-only, deliberately small subnet |
| 8 | CR14: after-hours contractors require limited wireless access, addressed via the DHCP design, preserving existing segmentation | 🟢 Brief §10 | Contractor segment must receive a DHCP-issued address and must not reach internal department resources | Contractor VLAN scoped by DHCP like any other VLAN; ACL denies contractor traffic toward Admin and Technical subnets, permits internet access (🟠 my own interpretation of "limited access," documented in the logical topology) |
| 9 | Deliberate configuration fault, documented, then resolved | 🟢 Brief §9 | Project must include one troubleshooting cycle with evidence | Planned fault: remove `ip helper-address` from R1's VLAN 20 sub-interface, producing a clean, isolated failure (VLAN 20 clients get no address, other VLANs keep working) to diagnose and resolve during implementation |
| 10 | At least one negative test with explanation | 🟢 Brief §9, §11 | Need at least one test case that is expected to fail | Denied file-share access attempt between Admin and Technical documented as the negative test |
| 11 | Assigned addressing block: 172.30.98.0/23 | 🟢 Brief §6, §16 | 512 total addresses to allocate across all VLANs | VLSM allocation, sized to each VLAN's actual need, see `addressing/ip-addressing-plan.md` |
| 12 | Employee counts, existing infrastructure, departmental structure, budget | Not stated in brief | No basis for exact figures | Explicitly left undefined; where a number is needed for subnet sizing, a conservative planning assumption is stated inline and flagged 🟠 |
| 13 | Comparable South African ISP offices commonly separate administrative and technical/NOC functions | 🟡 Industry-supported | Supports the two-department model chosen in #2 | Used only as supporting rationale, not as a factual claim about Kopano itself |

## 4. Client Problem Statement

Kopano requires a segmented telecommunications-office network that provides controlled
connectivity between different user groups while enforcing specific access restrictions.
The network must support VLAN segmentation, routed inter-VLAN communication, scoped DHCP
allocation across VLANs with relay, controlled printer access between departments,
restricted file sharing between departments, and a dedicated limited-access wireless
segment for after-hours contractors (CR14). I've drawn this directly from the brief's
stated requirements and constraint, without introducing any assumption about existing
network faults or performance problems that the brief never mentions.

## 5. Why These Decisions, Not Alternatives

The matrix above shows what I chose; this section shows why the alternative would fail.

- If the printer stayed inside Admin's VLAN instead of getting its own VLAN, Technical
  would need either a routed ACL exception carved specifically into Admin's VLAN, or its
  own separate printer, since VLANs by definition isolate broadcast domains from each
  other. Either option is messier than one shared VLAN both departments can reach
  identically, and the exception-based option quietly weakens Admin's isolation from
  Technical, which is the exact thing the constraint says must not happen.
- If DHCP stayed on the router instead of moving to a dedicated server, the router would
  never need to forward anything to itself, so no relay would ever occur and `giaddr` would
  never be populated. The brief's technical challenge specifically asks for DHCP "across
  VLAN boundaries using an appropriate relay/routing arrangement" (§9), so router-hosted
  DHCP would leave that clause undemonstrated, not just under-explained.
- If Admin and Technical shared one VLAN instead of two, "printer sharing must cross
  departments but file sharing must not" would have no meaning at all, since there would be
  no department boundary for anything to cross.
- If the contractor segment weren't isolated with its own VLAN and ACL rules, CR14's
  requirement to preserve "existing segmentation" while still granting contractors network
  access would be violated the moment a contractor device joined the network.

## 6. Explicitly Unresolved / Out of Scope

- Exact employee/device counts: not specified, not fabricated.
- Budget: not specified, not fabricated. Device selection prioritises technical suitability
  for the stated requirements over cost.
- Any Kopano infrastructure that may exist prior to this project: not specified, not
  assumed. I'm treating this as a greenfield build that satisfies the brief.

## 7. Course Concepts Applied in This Design

For quick cross-reference during the video defence:

| Concept | Where it's applied |
|---|---|
| Broadcast domains, Layer 2 vs Layer 3 | Logical topology §1: why 4 VLANs mean 4 broadcast domains, and why inter-VLAN traffic needs the router |
| Extended star / tree / hybrid topology, vs bus/ring/mesh | Physical topology §2 |
| Collision domains, why switches replaced hubs | Physical topology §2 |
| DORA (Discover, Offer, Request, Acknowledge) | Requirements matrix row 5; logical topology §4 |
| DHCP relay, `ip helper-address`, `giaddr` | Requirements matrix row 5; addressing plan §4; logical topology §4 |
| VLSM vs FLSM, borrow-bits/block-size method | Addressing plan §1 |
| Exclusion vs reservation | Addressing plan §3 |
| Standard vs extended ACLs | Logical topology §3 |
| NAT/PAT, private vs public address space | Physical topology §4 |
