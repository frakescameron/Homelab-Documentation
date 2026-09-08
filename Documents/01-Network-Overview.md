# Network Overview

## Purpose

This document provides a high-level summary of the network design, the reasoning behind key architectural decisions, and the current state of the build. Detailed configuration for each area is broken out into the other documents in this folder.

## Design Philosophy

The network is built around a few core principles:

- **Separation of duties** — routing, switching, and wireless are handled by dedicated devices (pfSense, the Cisco Catalyst switch, and the ASUS access point) rather than a single consumer all-in-one router. This mirrors how enterprise networks are typically built and gives more visibility and control at each layer.
- **Segmentation by trust level** — devices are grouped into VLANs based on their function and risk profile (wireless, server, and trusted workstation) rather than being left flat on one network.
- **Deny-by-default** — every VLAN starts with no rules (implicit deny) and only the traffic that is explicitly needed is allowed. Inter-VLAN access is the exception, not the default.
- **Document the "why," not just the "what"** — every major decision in this repository includes the reasoning behind it, since the reasoning is what actually gets evaluated, not just the end configuration.

## Current Architecture

```
Internet
    |
    v
Motorola MB7621 (Cable Modem)
    |
    v
pfSense Firewall (Router-on-a-Stick)
    |
    v
Cisco Catalyst WS-C2960CG-8TC-L (802.1Q Trunk)
    |
    +-- VLAN 10 -- AP / Wireless
    +-- VLAN 20 -- Homelab Server (Proxmox Host)
    +-- VLAN 30 -- Desktop PC / Trusted
```

pfSense performs all routing, firewalling, NAT, DHCP, and DNS resolution for the network. The Cisco switch handles Layer 2 switching and VLAN tagging. The ASUS access point operates purely as a wireless bridge (Access Point mode), with no routing or DHCP responsibilities of its own.

## Build Status

This network is being built incrementally, and this documentation reflects the current in-progress state:

| Component | Status |
|---|---|
| pfSense base configuration (interfaces, VLANs, DHCP) | Complete |
| Firewall rules per VLAN | Complete |
| WAN hardening | Complete |
| SSH key-based authentication | Complete |
| DNS Resolver (DNSSEC + DNS-over-TLS) | Complete |
| Suricata IDS | Running in detection-only mode, under evaluation |
| pfBlockerNG (DNSBL) | Complete |
| Cisco switch VLAN/trunk configuration | In progress |
| Automated configuration backups | Planned |

Until the switch is fully configured with the 802.1Q trunk, the access point is connected directly to the untagged LAN interface as an interim measure. See `04-Routing.md` for details on the router-on-a-stick design and `07-Future-Plans.md` for the switch configuration timeline.

## Related Documents

- `02-Ip-Addressing.md` — IP scheme and DHCP details
- `03-Vlans.md` — VLAN design and rationale
- `04-Routing.md` — Router-on-a-stick implementation
- `05-Firewall-Rules.md` — Firewall rule sets and reasoning
- `06-Services.md` — DNS, DHCP, IDS, and content filtering configuration
- `07-Future-Plans.md` — Planned upgrades and improvements
