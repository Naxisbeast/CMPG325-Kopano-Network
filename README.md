# CMPG325-Kopano-Network

Individual semester project for CMPG 325, Computer Networks (Advanced Project Level),
North-West University.

**Project ID:** CMPG325-2026-147 | **Client ID:** CLI-147
**Assigned Client:** Kopano Fibre & Wireless ISP (Mahikeng) | **Industry:** Telecommunications
**Student:** Wana, Kamo (52060586)

> Note to self: double-check this student number against my actual registration before
> final submission. A mismatch here is a zero-effort way to lose marks.

## Assigned Networking Challenge

DHCP: specifically, scoped multi-VLAN address assignment with relay across Layer 3
boundaries. A client's DHCPDISCOVER is a Layer 2 broadcast, so once the office network is
split into four VLANs (four separate broadcast domains), no client outside the VLAN
hosting the DHCP server can reach it without a relay agent forwarding that broadcast out of
its own segment. Getting that right, with correct exclusions, MAC-based reservations for
infrastructure, and a deliberately introduced then resolved configuration fault, is the
core of this project.

## Repository Structure

```
CMPG325-Kopano-Network/
├── README.md
├── requirements/
│   └── client-requirements.md      Requirements traceability matrix
├── topology/
│   ├── physical-topology.md        Device inventory, topology classification, diagrams
│   └── logical-topology.md         VLAN plan, ACL policy, DHCP relay flow
├── addressing/
│   └── ip-addressing-plan.md       VLSM breakdown of 172.30.98.0/23
└── docs/
    └── milestone-1.md              Milestone 1 summary and index
```

As the project progresses past Milestone 1, this repository will be extended with:

```
packet-tracer/     .pkt file(s)
configs/           Exported device configurations
dhcp/              DHCP-specific evidence and configuration notes
acl/               Access-control configuration and testing evidence
testing/           Connectivity and negative-test evidence
troubleshooting/   Deliberate fault, diagnosis, fix, and verification
video/             Link to the individual video demonstration
```

## Design Approach

I've traced every design decision in this repository to one of three sources, labelled
throughout: a stated requirement in the project brief, an industry-supported inference
about comparable telecommunications office environments, or my own explicit design
assumption where the brief doesn't specify a detail. I haven't presented anything about the
assigned client, Kopano Fibre & Wireless ISP, as established beyond what the brief states.

## Current Status

Milestone 1, Client Requirements & Network Design, is complete. See
[`docs/milestone-1.md`](docs/milestone-1.md).
