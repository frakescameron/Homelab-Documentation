# IP Addressing

## Addressing Scheme

Each VLAN is assigned its own /24 network. A /24 was chosen for every subnet, including the single-host server segment, for consistency and future headroom rather than tightly right-sizing each subnet (e.g. a /29 or /30). In an enterprise environment with constrained address space or route summarization requirements, right-sizing subnets would be the better practice — but for a homelab where RFC1918 space is effectively unlimited, standardizing on /24 keeps every subnet easy to remember and avoids ever having to resize a live network as it grows.

| VLAN | Name | Network | Usable Range | Gateway (pfSense) |
|---|---|---|---|---|
| — | LAN (legacy, pre-VLAN) | 192.168.1.0/24 | 192.168.1.2 – 192.168.1.254 | 192.168.1.1 |
| 10 | AP | 10.10.10.0/24 | 10.10.10.2 – 10.10.10.254 | 10.10.10.1 |
| 20 | SERVER | 10.10.20.0/24 | 10.10.20.2 – 10.10.20.254 | 10.10.20.1 |
| 30 | DESKTOP_PC | 10.10.30.0/24 | 10.10.30.2 – 10.10.30.254 | 10.10.30.1 |

The LAN network (192.168.1.0/24) remains in place as the original untagged network. It currently carries the access point's uplink until the switch trunk is configured, at which point the AP will move to its dedicated VLAN 10 subnet.

## DHCP

DHCP is served by pfSense using the **Kea DHCP** backend. pfSense migrated from the legacy ISC DHCP backend to Kea proactively, ahead of ISC DHCP's planned removal from future pfSense releases. See `06-Services.md` for details on that migration and a known limitation around DNS Resolver registration.

| Interface | DHCP Pool | Notes |
|---|---|---|
| AP (VLAN 10) | 10.10.10.10 – 10.10.10.244 | Dynamic pool for wireless clients |
| SERVER (VLAN 20) | Static assignment (no DHCP pool) | The Proxmox host is the only device on this VLAN and is statically addressed on the host itself rather than relying on a DHCP reservation |
| DESKTOP_PC (VLAN 30) | 10.10.30.10 – 10.10.30.244 | Dynamic pool for the primary workstation and any additional trusted devices |

## Reserved / Isolated Addressing

| Purpose | Address | Notes |
|---|---|---|
| pfBlockerNG DNSBL Virtual IP | 172.16.0.1 | Deliberately placed in an isolated 172.16.0.0/12 range that is not assigned to any interface, per pfBlockerNG's requirement that the DNSBL VIP not overlap with any in-use network |

## Notes

- Public/WAN-facing IP addresses are intentionally omitted from this documentation.
- All internal addressing shown here is RFC1918 private space and poses no external exposure risk on its own.
