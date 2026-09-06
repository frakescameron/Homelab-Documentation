# Routing

## Design: Router-on-a-Stick

pfSense's six physical Intel Gigabit NICs mean a dedicated physical interface per VLAN was an available option. Router-on-a-stick (a single trunked interface carrying multiple tagged VLANs) was chosen instead, for a few reasons:

- It mirrors how VLAN routing is actually done in the vast majority of real environments, where physical interfaces are far outnumbered by the VLANs that need routing between them. Practicing this design is more directly relevant to real-world networking work than relying on one-NIC-per-VLAN, which doesn't scale past a handful of segments.
- It leaves the remaining physical NICs on pfSense free for future use (a dedicated WAN failover interface, an out-of-band management interface, or additional physical segments) rather than consuming them for VLANs that can be handled with tagging instead.
- It requires a proper 802.1Q trunk to the switch, which is directly relevant CCNA/switching material to practice hands-on.

## Implementation

- **Physical trunk interface:** `igb1`
- **VLAN sub-interfaces:** `igb1.10` (AP), `igb1.20` (SERVER), `igb1.30` (DESKTOP_PC)
- Each VLAN sub-interface is assigned its own IP address (the gateway for that subnet — see `02-Ip-Addressing.md`) and is treated by pfSense as an independent interface for firewall rule purposes.

```
pfSense (igb1)
    |
    | 802.1Q Trunk (VLANs 10, 20, 30)
    v
Cisco Catalyst Switch
    |
    +-- Access Port (VLAN 10) --> AP
    +-- Access Port (VLAN 20) --> Proxmox Server
    +-- Access Port (VLAN 30) --> Desktop PC
```

## Inter-VLAN Routing

All routing between VLANs happens on pfSense itself, since each VLAN is a directly connected subnet on the firewall. No dynamic routing protocol is in use — with only one router in the network, static/directly-connected routing is sufficient and a dynamic routing protocol (OSPF, etc.) would add complexity with no practical benefit at this scale. This is called out explicitly because a network engineering interview may ask when dynamic routing becomes necessary — the honest answer here is: not yet, at this scale, with a single router.

## NAT

Outbound NAT is configured on the WAN interface using pfSense's default automatic outbound NAT rules, translating all internal VLAN traffic to the WAN IP for internet access. No port forwarding / inbound NAT rules are currently configured, consistent with the WAN firewall policy described in `05-Firewall-Rules.md` (no inbound connections are permitted from the internet).

## Current Limitation

Because the switch side of the trunk is not yet configured (see `07-Future-Plans.md`), inter-VLAN routing is only exercised for the SERVER and DESKTOP_PC VLANs, which are reachable from directly connected test hosts on those interfaces. The AP VLAN's routing/firewall rules are configured and validated on pfSense but not yet exercised by real wireless traffic.
