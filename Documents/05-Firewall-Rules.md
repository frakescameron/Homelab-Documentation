# Firewall Rules

## Philosophy

Every interface on pfSense starts with an implicit deny — no traffic passes until a rule explicitly allows it. Rather than writing a single broad "allow any to any" rule per VLAN, each VLAN's ruleset was built around the specific question: **what does this VLAN actually need to reach, and what should it explicitly never reach?** Anything not explicitly required is denied by default.

Rule order matters in pfSense — rules are evaluated top to bottom and the first match wins. Every block rule for a given VLAN is placed above that VLAN's allow-any rule, so that the specific denies are always evaluated before the general internet-access allow.

## Aliases

To keep rules readable and maintainable, IP Aliases were created for each VLAN's subnet rather than typing raw CIDR ranges into every rule. This also means the network can be re-addressed later by editing the alias once, rather than hunting through every rule that references it.

| Alias | Type | Value |
|---|---|---|
| AP_NET | Network(s) | 10.10.10.0/24 |
| SERVER_NET | Network(s) | 10.10.20.0/24 |
| DESKTOP_NET | Network(s) | 10.10.30.0/24 |
| RFC1918 | Network(s) | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 |

## WAN Rules

| Rule | Source | Destination | Action | Purpose |
|---|---|---|---|---|
| Block private networks | RFC1918 | Any | Block | Drops spoofed or misrouted traffic claiming to originate from private address space — this should never legitimately appear on WAN |
| Block bogon networks | Reserved/unallocated | Any | Block | Drops traffic from IP ranges that are unallocated by IANA and should never appear as a legitimate source on the internet |

No inbound allow rules exist on WAN. This means no service is exposed to the internet, and the pfSense web administration interface and SSH are only reachable from internal networks. If a service needs to be exposed publicly in the future, a narrowly-scoped rule plus a corresponding NAT entry would be added for that specific purpose only.

## AP VLAN (VLAN 10) Rules

The AP VLAN is treated as the least trusted segment, since wireless and IoT-adjacent devices are the most likely point of initial compromise on a home network.

| # | Source | Destination | Action | Purpose |
|---|---|---|---|---|
| 1 | AP_NET | DESKTOP_NET | Block | Prevents wireless clients from reaching the trusted workstation |
| 2 | AP_NET | SERVER_NET | Block | Prevents wireless clients from reaching the homelab server |
| 3 | AP_NET | Any | Allow | Permits internet access for wireless clients |

## SERVER VLAN (VLAN 20) Rules

The server VLAN hosts self-hosted services and is treated as sensitive. It has no need to reach any other internal VLAN, only the internet (for updates).

| # | Source | Destination | Action | Purpose |
|---|---|---|---|---|
| 1 | SERVER_NET | DESKTOP_NET | Block | The server has no legitimate reason to initiate a connection toward the workstation |
| 2 | SERVER_NET | AP_NET | Block | The server has no legitimate reason to initiate a connection toward wireless clients |
| 3 | SERVER_NET | Any | Allow | Permits outbound internet access for updates and any services that require it |

## DESKTOP_PC VLAN (VLAN 30) Rules

The desktop workstation is the most trusted segment and is the device actively used to manage and consume homelab services, so it is given one deliberate exception to reach the server VLAN.

| # | Source | Destination | Action | Purpose |
|---|---|---|---|---|
| 1 | DESKTOP_NET | AP_NET | Block | The workstation has no standing need to reach the wireless VLAN directly |
| 2 | DESKTOP_NET | SERVER_NET | Allow | **Deliberate exception** — allows the workstation to reach self-hosted services (e.g. Nextcloud, SSH, dashboards) running on the Proxmox host |
| 3 | DESKTOP_NET | Any | Allow | Permits general internet access |

### Why Desktop → Server Is the Only Exception

This is the one deviation from strict VLAN isolation in the network, and it was made deliberately rather than by omission. The alternative designs considered were:

- **Full isolation between all three VLANs** — simplest and most secure, but makes the homelab server's services unusable from the daily workstation without a workaround (e.g. jumping through a VPN for every internal request), which isn't a realistic trade-off for a home environment.
- **Allowing Server → Desktop instead** — rejected, because it would mean the more sensitive segment (server) could initiate connections toward the workstation, which is the wrong direction of trust for a segment hosting externally-reachable or less-hardened services.
- **Desktop → Server only (chosen)** — the workstation, the most trusted and actively-monitored device on the network, is allowed to initiate toward the server. The server itself still cannot initiate anything toward the workstation or the AP VLAN. This keeps the sensitive segment's blast radius contained while preserving day-to-day usability.

## Logging

Firewall rule logging is a planned addition (see `07-Future-Plans.md`) so that denied traffic across VLAN boundaries can be reviewed in `Status > System Logs > Firewall` rather than only being silently dropped.
