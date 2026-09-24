[⬅ Back to the main README](../README.md)

# Physical Topology
**Project:** CMPG325-2026-147, Kopano Fibre & Wireless ISP (Mahikeng)

## 1. Design Rationale

I scoped this as a small ISP office LAN, not Kopano's customer-facing fibre/wireless
infrastructure. The brief asks for the client's network environment (§2, §4), so I kept the
scope to the internal office rather than the ISP's last-mile network. I kept the device
count to the minimum that can still demonstrate every required technical challenge
(multi-VLAN, scoped and relayed DHCP, inter-VLAN ACLs, a wireless contractor segment).
Adding more devices than that would only add configuration and testing surface without
adding evidence that any requirement is actually being met. 🟠 Design decision.

## 2. Topology Classification

This design uses an extended star topology. Each switch, on its own, is already a star: a
central device with every connected node cabled directly to it, which is the standard
definition and the reason switches replaced hubs in the first place. A hub forwards every
incoming frame out every port, so every device on it shares one collision domain; a switch
gives each port its own collision domain and only forwards a frame out the port it's
actually destined for. That's why star wiring around a switch, not a bus around a shared
cable, is the sensible physical building block here.

I then take that building block and repeat it one level up: the access switches (SW-ADMIN,
SW-TECH, and the WRT300N AP) each form their own star around their connected end devices, and those
switches connect back to one central point, the core switch, which connects up to the
router. A star built out of stars like this is a hierarchical or tree topology, and it's
sometimes called a hybrid topology since it combines the star pattern at two levels. That
more precise term is the one I use in the diagram section below.

**Why not the other standard topologies:**

- Bus. A single shared backbone cable with all devices tapped onto it. I rejected this
  because one cable failure takes down the whole segment, there's no natural point to
  enforce VLAN or ACL boundaries, and it doesn't scale cleanly to four separate logical
  segments.
- Ring. Each device connects to exactly two neighbours, forming a loop. I rejected this
  because a single link or device failure can disrupt the whole ring, unless a second
  counter-rotating ring is added, which adds cost this project doesn't need, and it offers
  no advantage for a segmented office LAN.
- Full mesh. Every device connects directly to every other device. I rejected this because
  it needs far more cabling and ports than this office requires: with roughly 7 major
  devices, a full mesh would need 21 links for no benefit, since none of the requirements
  call for that level of redundancy.
- Star, single switch. One switch with every device on it. I rejected this on its own
  because a single switch can't cleanly host separate wiring closets per department while
  keeping the physical layout traceable to the VLAN plan, but it's the building block the
  extended star design below is made from.

**Why extended star fits this project:** it isolates a fault to one branch, so a failure on
SW-TECH doesn't take down Admin or the WRT300N AP; it gives each department its own physical access
point that maps cleanly onto its VLAN; and it's the standard real-world choice for exactly
this kind of segmented office LAN.

**How it could change if requirements changed:** if Kopano's brief had asked for redundant
links, so that no single switch failure could isolate a department, I'd move toward a
partial mesh between the core and access switches, or toward two core switches with
redundant uplinks. That isn't required here, since the brief doesn't ask for fault-tolerant
physical redundancy, so I haven't added it. I'm noting it anyway to show the topology choice
was deliberate for this brief, not the only topology that could ever apply.

## 3. Link Types

Every wired link in this design (Kopano-Edge-R1 to Kopano-Core-SW1, Kopano-Core-SW1 to each access switch, access
switches to end devices and the printer) is standard Gigabit Ethernet copper, which is more
than sufficient for an office of this size and is the default Packet Tracer link type for
these device classes. 🟠 Design decision, since the brief doesn't specify link speeds; I'm
stating it here so the physical topology is fully specified rather than leaving cabling
implicit. The contractor segment is wireless by requirement (CR14), using the WRT300N's default
802.11 configuration.

## 4. Devices

| Device | Role | Justification |
|---|---|---|
| 1x Router (Kopano-Edge-R1) | Inter-VLAN routing (router-on-a-stick), DHCP relay agent, NAT/PAT for internet-bound traffic | A single router-on-a-stick is sufficient for 4 VLANs; a Layer 3 switch would add capability this project's scale doesn't require, so I kept it simple, matching the rubric's "appropriate" rather than "maximal" |
| 1x Core/Distribution Switch (Kopano-Core-SW1) | Trunk aggregation from access switches and AP to the router | Central point for 802.1Q trunks |
| 2x Access Switches (SW-ADMIN, SW-TECH) | Access-layer connectivity for Admin and Technical end devices, and the shared printer | One per department keeps the physical wiring-closet story simple and traceable to the VLAN plan |
| 1x Wireless Access Point (WRT300N, L2-bridged) | Contractor Wi-Fi (CR14) | Explicitly required: "dedicated limited-access wireless segment" (Brief §9). Built as a pure Layer 2 bridge — WAN port bypassed, uplink on a LAN port, internal DHCP disabled, 802.1Q tagging handled by `Kopano-Core-SW1` (see README §4) |
| PCs (PC0-PC2 Admin, PC3-PC5 Technical) | End-user devices, Admin and Technical | Represent departmental staff |
| 1x Network Printer (Printer0) | Shared print device | Central to the printer-sharing requirement (Brief §8) |
| 1x DHCP Server (172.30.98.2) | Hosts the DHCP service, placed in VLAN 10 (Admin) | 🟠 Design decision: I moved DHCP off the router and onto a dedicated server so that relay is genuinely required for VLANs 20, 30, and 40, which is what the brief's technical challenge actually asks me to demonstrate. See `addressing/ip-addressing-plan.md` §4 for the full reasoning. |

**On NAT/PAT:** the ACL policy permits contractor devices to reach the internet, and every
VLAN's clients will need internet access for normal operation. Since 172.30.x.x is private
address space, none of it is routable on the public internet, so the router performs NAT
(specifically PAT, Port Address Translation, overloading the internal 172.30.98.0/23 block
onto the single public address 203.0.113.1) on its WAN-facing interface for traffic leaving
the network. This isn't part of the
assigned technical challenge, but it's necessary for the design to actually work end to end,
so I'm stating it here rather than leaving it implicit.

## 5. Physical Diagram (Extended Star)

```mermaid
graph TD
    WAN["ISP / WAN Uplink<br/>203.0.113.0/30<br/>ISP-Upstream .2 / DNS 8.8.8.8"] --> R1["Kopano-Edge-R1<br/>Router-on-a-Stick<br/>DHCP relay + NAT/PAT<br/>203.0.113.1"]
    R1 --> CORE["Kopano-Core-SW1"]
    CORE --> SWADMIN["Access Switch<br/>SW-ADMIN"]
    CORE --> SWTECH["Access Switch<br/>SW-TECH"]
    CORE --> AP["WRT300N Wireless AP<br/>(L2 bridge)<br/>Contractor Wi-Fi"]

    SWADMIN --> AdminPCs["Admin PCs PC0-PC2<br/>VLAN 10"]
    SWADMIN --> DHCPServer["DHCP Server 172.30.98.2<br/>VLAN 10"]
    SWADMIN --> Printer["Printer0 172.30.99.2<br/>VLAN 30"]
    SWTECH --> TechPCs["Technical PCs PC3-PC5<br/>VLAN 20"]
    AP --> Contractors["Laptop1-Laptop2<br/>VLAN 40"]

    style R1 fill:#dbeafe
    style CORE fill:#dbeafe
    style AdminPCs fill:#dcfce7
    style DHCPServer fill:#dcfce7
    style TechPCs fill:#fef3c7
    style Printer fill:#fce7f3
    style Contractors fill:#fee2e2
```

*(ASCII version below for quick reference outside GitHub's renderer.)*

```
                          ISP / WAN Uplink
                    (203.0.113.0/30, ISP-Upstream .2,
                     external DNS 8.8.8.8)
                                    |
                          [ Kopano-Edge-R1 ]
                          (Router-on-a-Stick, DHCP relay,
                           NAT/PAT outbound to 203.0.113.1)
                                    |
                           [ Kopano-Core-SW1 ]
                                    |
              ┌─────────────────────┼─────────────────────┐
              |                     |                     |
       [ SW-ADMIN ]          [ SW-TECH ]          [ WRT300N ]
          |    |    |            |    |              (L2 bridge)
       PC0-PC2  DHCP         PC3-PC5               Laptop1
       (Admin,  Server        (Tech,                Laptop2
        VLAN 10) 172.30.98.2   VLAN 20)              (VLAN 40)
                  (VLAN 10)
          |
       Printer0
       172.30.99.2
         (VLAN 30)
```

Both the printer and the DHCP server plug into the SW-ADMIN access switch physically, but
neither one belongs to VLAN 10 by default just because of where it's cabled: the printer's
access port is assigned to VLAN 30 and the DHCP server's access port stays in VLAN 10. This
is the concrete demonstration that physical wiring (Layer 1) and logical VLAN membership
(Layer 2) are separate decisions: a device's cable doesn't decide its broadcast domain, its
switch-port configuration does. That separation is exactly what makes "printer sharing
crosses departments, file sharing doesn't" achievable with a clean ACL instead of
per-exception rules bolted onto each department's VLAN.

## 6. Trunk / Access Port Summary

| Link | Type | VLANs Carried |
|---|---|---|
| Kopano-Edge-R1 to Kopano-Core-SW1 | Trunk (802.1Q) | 10, 20, 30, 40 |
| Kopano-Core-SW1 to SW-ADMIN | Trunk | 10, 30 |
| Kopano-Core-SW1 to SW-TECH | Trunk | 20 |
| Kopano-Core-SW1 to WRT300N | Access | 40 |
| SW-ADMIN to Admin PCs (PC0-PC2) | Access | 10 |
| SW-ADMIN to DHCP Server (172.30.98.2) | Access | 10 |
| SW-ADMIN to Printer0 (172.30.99.2) | Access | 30 |
| SW-TECH to Technical PCs (PC3-PC5) | Access | 20 |

I made the Kopano-Core-SW1-to-WRT300N link an access port rather than a trunk, deliberately.
The WRT300N carries exactly one VLAN (VLAN 40, confirmed in the logical topology, where it's
the only scope relayed to the AP), so trunking here would tag every frame with a VLAN ID
that can only ever take one value, which is functionally pointless. The access-switch
uplinks, by contrast, are all 802.1Q trunks: Kopano-Edge-R1 to Kopano-Core-SW1 carries all
four VLANs, Kopano-Core-SW1 to SW-ADMIN carries 10 and 30, and Kopano-Core-SW1 to SW-TECH
carries only 20, since SW-TECH hosts nothing but the Technical PCs (VLAN 20). Keeping
SW-TECH's uplink trunked is deliberate even though it needs only one VLAN today, so adding a
VLAN to that closet later wouldn't require a port-mode change. The contrast that motivated
the access port is unchanged: a link carrying exactly one VLAN gains nothing from 802.1Q
tagging; a link carrying several VLANs requires it.
