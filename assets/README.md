[⬅ Back to the main README](../README.md)

# Assets

Screenshot evidence for the test matrix and the troubleshooting cycle.

## `evidence/` — Test Evidence (TEST-01..TEST-06)

| Test ID | Filename | Capture |
| --- | --- | --- |
| TEST-01 | `evidence/test01_dhcp_lease.png` | `ipconfig /all` on PC3 showing `172.30.98.130/25` and GW `172.30.98.129` |
| TEST-02 | `evidence/test02_nat_ping.png` | Successful `ping 8.8.8.8` from PC0, 0% packet loss |
| TEST-03 | `evidence/test03_printer_ping.png` | Successful `ping 172.30.99.2` from PC3 (permitted by ACL 102) |
| TEST-04 | `evidence/test04_ftp_block.png` | FTP attempt from PC3 to `172.30.98.2` timing out (ACL 102 match hits) |
| TEST-05 | `evidence/test05_guest_block.png` | `ping 172.30.98.2` from Laptop1 → Destination Host Unreachable (ACL 100) |
| TEST-06 | `evidence/test06_ssh_verify.png` | SSHv2 session from PC1 to `172.30.98.1` authenticated as `admin` |

## `faults/` — Troubleshooting Cycle (FLT-01..FLT-03)

| Fault ID | Filename | Capture |
| --- | --- | --- |
| FLT-01 (DHCP relay) | `faults/fault-01a-dhcp-apipa.png` | PC3 showing APIPA `169.254.x.x` after `ip helper-address` removed |
|  | `faults/fault-01b-dhcp-recovered.png` | PC3 showing restored lease `172.30.98.130/25` |
| FLT-02 (trunk pruning) | `faults/fault-02a-vlan20-pruned-ping-fail.png` | PC3 ping to gateway timing out (100% loss) |
|  | `faults/fault-02b-vlan20-trunk-restored.png` | PC3 ping to gateway successful (0% loss) |
| FLT-03 (ACL over-block) | `faults/fault-03a-acl-overblock-ping.png` | PC3 ping to `172.30.98.2` → Destination Host Unreachable |
|  | `faults/fault-03b-acl-restored-ping-pass.png` | PC3 ICMP to printer restored; FTP to server still blocked |

Referenced from [`README.md`](../README.md) §4 and §5, and [`docs/troubleshooting-log.md`](../docs/troubleshooting-log.md).
