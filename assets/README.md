[⬅ Back to the main README](../README.md)

# Assets

Screenshot evidence for the test matrix and the troubleshooting cycle.

## `evidence/` — Test Evidence (TEST-01..TEST-10)

| Test ID | Filename | Capture |
| --- | --- | --- |
| TEST-01 | `evidence/test-01-dhcp-pc3.png` | `ipconfig /all` on PC3 showing `172.30.98.132/25`, GW `172.30.98.129`, DNS `8.8.8.8` |
| TEST-02 | `evidence/test-02-nat-pc0.png` | `ping -t 8.8.8.8` from PC0 side-by-side with `show ip nat translations` on `Kopano-Edge-R1` |
| TEST-03 | `evidence/test-03-printer-ping.png` | Successful `ping 172.30.99.2` from PC3 (permitted by ACL 102) |
| TEST-04 | `evidence/test-04-ftp-block-pc3.png` | `ftp 172.30.98.2` from PC3 timing out, with `show access-lists 102` match counters |
| TEST-05 | `evidence/test-05-guest-block-contractor-1.png` | `ping 172.30.98.2` from Contractor-1 timing out, with `show access-lists 100` hits (ACL 100) |
| TEST-06 | `evidence/test-06-ssh-pc0.png` | SSHv2 session from PC0 to `172.30.98.1` as `KopanoAdmin`, landing at `Kopano-Edge-R1>` |
| TEST-07 | `evidence/test-07-dhcp-vlan10-pc0.png` | `ipconfig /all` on PC0 showing valid Admin VLAN 10 lease + DNS `8.8.8.8` |
| TEST-10 | `evidence/test-10-contractor-internet.png` | `ping 8.8.8.8` from Contractor-1, 0% loss (contractor Internet while internal stays blocked) |

## `faults/` — Troubleshooting Cycle (FLT-01..FLT-03)

| Fault ID | Filename | Capture |
| --- | --- | --- |
| FLT-01 (DHCP relay) | `faults/flt-01-dhcp-fail.png` | PC3 showing APIPA `169.254.x.x` after `ip helper-address` removed |
|  | `faults/flt-01-dhcp-recover.png` | PC3 showing restored lease `172.30.98.132/25` |
| FLT-02 (trunk pruning) | `faults/flt-02-trunk-fail.png` | PC4 ping to gateway timing out (100% loss) |
|  | `faults/flt-02-trunk-recover.png` | PC4 ping to gateway successful (0% loss) |
| FLT-03 (ACL over-block) | `faults/flt-03-acl-fail.png` | PC5 ping to `172.30.98.2` → Destination Host Unreachable |
|  | `faults/flt-03-acl-recover.png` | PC5 ICMP to printer restored; FTP to server still blocked |

Referenced from [`README.md`](../README.md) §4 and §5, and [`docs/troubleshooting-log.md`](../docs/troubleshooting-log.md).
