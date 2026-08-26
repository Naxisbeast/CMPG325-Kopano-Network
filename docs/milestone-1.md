# Milestone 1 - Client Design Review
**Project:** CMPG325-2026-147 | **Client:** Kopano Fibre & Wireless ISP (Mahikeng)
**Student:** Wana, Kamo (52060586) | **Due:** 28 August 2026

## Purpose

This milestone answers one question: if Kopano Fibre & Wireless ISP hired me as a network
engineer, what network am I proposing, why does it satisfy their stated requirements, and
how have I planned the addressing? This is a design review, not a configuration review, so
no Packet Tracer build, DHCP configuration, or ACL commands appear yet; those follow in the
implementation phase.

## What This Milestone Contains

1. **[Client Requirements](../requirements/client-requirements.md)**: full traceability
   matrix mapping every design decision back to the brief, with assumptions explicitly
   labelled (🟢 brief fact, 🟡 industry-supported, 🟠 design assumption), plus a "why not
   the alternative" section and a course-concepts cross-reference.
2. **[Physical Topology](../topology/physical-topology.md)**: device inventory, topology
   classification against the taught topology types, and a trunk/access port plan that
   includes the AP wired as an access port in VLAN 40.
3. **[Logical Topology](../topology/logical-topology.md)**: VLAN plan, inter-VLAN ACL
   policy, and the DHCP relay logical flow.
4. **[IP Addressing Plan](../addressing/ip-addressing-plan.md)**: VLSM breakdown of
   172.30.98.0/23 with the subnetting method shown and a sanity check on the /24 boundary,
   DHCP scope summary, DHCP server placement with relay configuration, and what
   `ip helper-address` forwards by default.

## Design Summary

| Element | Decision |
|---|---|
| VLANs | 10 (Admin), 20 (Technical/NOC), 30 (Printers), 40 (Contractor Wireless) |
| Topology classification | Extended star (hierarchical/tree); bus, ring, and full mesh each considered and rejected with reasons |
| Addressing | VLSM: two /25s (Admin, Technical), two /28s (Printers, Contractor) within 172.30.98.0/23; the two /25s together span the first /24 of the block exactly, no gap or overlap |
| Routing | Router-on-a-stick, single router, four 802.1Q sub-interfaces |
| DHCP | One server in VLAN 10, four scopes, relayed via `ip helper-address` on the VLAN 20/30/40 sub-interfaces; VLAN 10's own scope is matched by the arriving subnet, with no relay and no giaddr |
| Access control | Admin/Technical to Printers: permit. Admin to Technical: deny. Contractor to Admin/Technical: deny. Contractor to Internet: permit |

## How This Satisfies the Assigned Challenge

The brief's advanced networking challenge is DHCP with scoped multi-VLAN address
assignment (§9). This design gives every VLAN its own scope, correct per-VLAN default
gateways, and documented exclusions and reservations for static infrastructure. Because the
DHCP server sits in VLAN 10 rather than on the router itself, VLANs 20, 30, and 40 all
genuinely need `ip helper-address` to reach it: their DHCPDISCOVER broadcasts would
otherwise never leave their own broadcast domain. In those relayed VLANs, giaddr selects
the scope; in VLAN 10, where the server shares the client's broadcast domain and no relay
runs, the scope is matched to the subnet the request physically arrived on instead. That's
what turns "relay across VLAN boundaries" from a described mechanism into a demonstrated
one, and it's traced clause by clause in the requirements matrix.

## How This Satisfies the Constraint and Change Request

- **Constraint (§8):** printer sharing crosses departments, file sharing doesn't. Solved
  with a dedicated Printer VLAN and a two-rule ACL pattern instead of per-exception rules.
- **CR14 (§10):** contractor wireless access. Solved with a dedicated, deliberately small
  Contractor VLAN, DHCP-scoped like every other VLAN, with access limited to the internet.
  I'm treating that internet-only scope as my own reading of "limited access" since the
  brief doesn't spell out the exact permitted resources; this is flagged as a design
  assumption in the logical topology document, not presented as a brief requirement.

## What Is Deliberately Not Yet Decided

Exact employee counts, existing Kopano infrastructure, and budget are not specified in the
brief, and I haven't invented them. Where a planning number was needed, for example subnet
sizing for Admin and Technical, I've stated and labelled the assumption in the addressing
plan rather than presenting it as fact.

## Next Steps (Post-Milestone-1)

1. Build the topology in Cisco Packet Tracer.
2. Configure VLANs, trunking, router sub-interfaces, and the DHCP server's scopes.
3. Configure and verify `ip helper-address` relay on the VLAN 20, 30, and 40 sub-interfaces.
4. Implement and test the ACL policy above, including at least one negative test.
5. Introduce the planned fault (remove `ip helper-address` from R1's VLAN 20
   sub-interface), document the diagnosis, then resolve it with evidence.
6. Capture verification evidence: DHCP leases, scope info, gateway settings, ACL behaviour.
