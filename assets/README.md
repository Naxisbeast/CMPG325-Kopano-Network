# Evidence Screenshots

Screenshot evidence referenced from the [Test Evidence & Verification Matrix](../README.md#5-test-evidence--verification-matrix).

Drop the screenshots into this folder using the exact filenames below so the links in the README resolve:

| Test ID | Filename | Suggested capture |
| --- | --- | --- |
| TEST-01 | `test01_dhcp_lease.png` | `ipconfig /all` on PC3 showing `172.30.98.130/25` and GW `172.30.98.129` |
| TEST-02 | `test02_nat_ping.png` | Successful `ping 8.8.8.8` from PC0 with 0% packet loss |
| TEST-03 | `test03_printer_ping.png` | Successful `ping 172.30.99.2` from PC3 (permitted by ACL 102) |
| TEST-04 | `test04_ftp_block.png` | FTP attempt from PC3 to `172.30.98.2` timing out (ACL 102 match hits) |
| TEST-05 | `test05_guest_block.png` | `ping 172.30.98.2` from Laptop1 returning Destination Host Unreachable (ACL 100) |
| TEST-06 | `test06_ssh_verify.png` | SSHv2 session from PC1 to `172.30.98.1` authenticated as `admin` |
