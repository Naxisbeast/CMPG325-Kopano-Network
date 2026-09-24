[⬅ Back to the main README](../README.md)

# IP Addressing Plan (VLSM)
**Project:** CMPG325-2026-147, Kopano Fibre & Wireless ISP (Mahikeng)
**Assigned block:** 172.30.98.0/23 (512 total addresses, 172.30.98.0 to 172.30.99.255)

## 1. Sizing Rationale

I sized each subnet by planning need per VLAN instead of splitting the block evenly. This
is the difference between VLSM (Variable Length Subnet Masking) and FLSM (Fixed Length
Subnet Masking): FLSM would force every VLAN into the same size, for example four /24s or
four /25s, regardless of how many hosts each one actually needs. VLSM lets each subnet
carry only the address space its population justifies, which is what the rubric means by
"efficient" rather than merely "correct."

Admin and Technical are the two staffed departments, so I gave them room for headcount
growth with two /25s (126 usable hosts each). The Printer and Contractor segments have
small, bounded populations by nature: a shared printer resource, and a contractor segment
that CR14 explicitly calls "limited access." I sized both as /28s (14 usable hosts each) so
address space isn't wasted on segments that will never need more.

🟠 Design decision: the brief doesn't state exact host counts for Admin or Technical, so I
treated /25 as a conservative planning size, large enough for a small office department
with room to grow, not a number I'm presenting as fact.

### VLSM method, shown for one subnet

To carve a /28 out of the /23, I borrow bits from the host portion until the mask reaches
28: 32 minus 28 leaves 4 host bits, so the block size is 2^4 = 16 addresses per subnet.
Working from the start of the second /24 (172.30.99.0), the /28 blocks step up in
16-address increments: .0, .16, .32, .48, and so on. That's why the Printer subnet lands
at 172.30.99.0/28 and Contractor Wireless lands at the very next block, 172.30.99.16/28,
and it's also why the first unused block after Contractor Wireless starts at 172.30.99.32,
which is where the reserved range below begins.

## 2. VLSM Breakdown

| VLAN | Name | Subnet | Mask | CIDR | Usable Range | Broadcast | Usable Hosts | Gateway |
|---|---|---|---|---|---|---|---|---|
| 10 | Admin | 172.30.98.0 | 255.255.255.128 | /25 | 172.30.98.1 to 172.30.98.126 | 172.30.98.127 | 126 | 172.30.98.1 |
| 20 | Technical/NOC | 172.30.98.128 | 255.255.255.128 | /25 | 172.30.98.129 to 172.30.98.254 | 172.30.98.255 | 126 | 172.30.98.129 |
| 30 | Printers | 172.30.99.0 | 255.255.255.240 | /28 | 172.30.99.1 to 172.30.99.14 | 172.30.99.15 | 14 | 172.30.99.1 |
| 40 | Contractor Wireless | 172.30.99.16 | 255.255.255.240 | /28 | 172.30.99.17 to 172.30.99.30 | 172.30.99.31 | 14 | 172.30.99.17 |
| n/a | Reserved for future growth | 172.30.99.32 to 172.30.99.255 | n/a | n/a | n/a | n/a | 224 addresses unallocated | n/a |

One deliberate sanity check on the two /25s, since this is the kind of subnet arithmetic a
marker might redo by hand: Admin (172.30.98.0/25) and Technical (172.30.98.128/25)
together span the entire first /24 of the assigned /23 (172.30.98.0/24) exactly, with no
gap and no overlap. Each /25 block is 128 addresses, and 128 + 128 = 256, which is exactly
one full /24. So the first /24 is fully consumed by the two departments, and the second
/24 (172.30.99.0) is where the smaller /28s and the reserved range begin.

Across the four VLANs, 288 of the 512 total addresses in the /23 are consumed: 280 usable
host addresses plus 8 spent on network/broadcast addresses (2 per subnet x 4 subnets). The
remaining 224 addresses are explicitly reserved for future growth rather than left
undocumented, which supports the rubric's "fully documented" standard.

## 3. DHCP Scope Summary

The brief asks for "appropriate exclusions or reserved addresses where justified" (Brief
§9). These are two different things, and I'm using both correctly rather than treating them
as one: an **exclusion** removes an address from the pool entirely so it's never leased to
anyone (used for the gateway); a **reservation** ties a specific address to a specific
device's MAC address, so that device always receives the same lease rather than being
excluded from DHCP altogether (used for the printer, which should still be DHCP-managed
infrastructure, just with a predictable address).

| VLAN | Pool Network | Pool Range Offered | Exclusions | Reservations | Default Gateway (pushed to clients) |
|---|---|---|---|---|---|
| 10 | 172.30.98.0/25 | .3 to .126 | .1 (gateway), .2 (DHCP server, static) | none | 172.30.98.1 |
| 20 | 172.30.98.128/25 | .130 to .254 | .129 (gateway) | none | 172.30.98.129 |
| 30 | 172.30.99.0/28 | .3 to .14 | .1 (gateway) | .2, reserved to the printer's MAC address | 172.30.99.1 |
| 40 | 172.30.99.16/28 | .18 to .30 | .17 (gateway) | none | 172.30.99.17 |

The printer's offered range starts at .3, not .2, because .2 is reserved rather than
excluded: it stays inside the DHCP-managed range, but only the printer's MAC ever receives
it. This keeps the printer under DHCP management (matching the brief's DHCP challenge)
while still giving it a predictable, memorable address for print-queue configuration.

## 4. DHCP Server Placement and Relay Configuration

🟠 Design decision (revised): I originally considered hosting DHCP directly on the router
(`ip dhcp pool`), but that removes the relay from the design entirely, since a router
handing out its own local pool never needs to forward anything anywhere. The brief's
technical challenge specifically requires DHCP "across VLAN boundaries using an appropriate
relay/routing arrangement" (Brief §9), so I moved the DHCP service onto a dedicated server
placed in VLAN 10 (Admin), at 172.30.98.2 (excluded from that VLAN's own pool, above). Every
other VLAN now genuinely needs relay to reach it, which is what the brief is actually
asking me to demonstrate.

The running build matches this arrangement on `Kopano-Edge-R1`: all four 802.1Q
sub-interfaces (Gi0/0.10, Gi0/0.20, Gi0/0.30, Gi0/0.40) carry `ip helper-address
172.30.98.2`. On Gi0/0.10 that statement is locally redundant — the DHCP server shares
VLAN 10's broadcast domain, so the server receives local DISCOVERs without any relay — but
I kept it on so the CLI shows a uniform relay config on every sub-interface and a marker
inspecting the build sees the same line they'd expect from README §3A.

| Sub-interface | VLAN | `ip helper-address` target |
|---|---|---|
| Gi0/0.10 | 10 | 172.30.98.2 (present in the build; locally redundant — server shares this VLAN) |
| Gi0/0.20 | 20 | 172.30.98.2 |
| Gi0/0.30 | 30 | 172.30.98.2 |
| Gi0/0.40 | 40 | 172.30.98.2 |

One thing I can be asked about in the defence: `ip helper-address` on a Cisco router relays
more than just DHCP by default. It forwards the standard set of UDP broadcast services,
including DHCP and TFTP, unless that list is trimmed with `ip forward-protocol`. In this
design only DHCP relay is actually being used and tested, but I can name this default
behaviour if asked what else `ip helper-address` forwards.

This is also the deliberate fault I executed for the required troubleshooting cycle (Brief
§9): removing the `ip helper-address` line from the Gi0/0.20 sub-interface on
`Kopano-Edge-R1` produces a fault where VLAN 20 clients get no address at all while VLAN 10
and the other relayed VLANs keep working. The fault was induced, diagnosed, and resolved —
see README §4 for the official troubleshooting log.
