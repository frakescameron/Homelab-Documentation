# VLANs

## Purpose of Segmentation

Prior to this build, every device on the network — the access point, the homelab server, and the primary workstation — shared a single flat LAN. Flat networks mean any compromised or misbehaving device can freely reach every other device. VLANs were introduced to segment the network by function and trust level, so that a compromise or misconfiguration in one segment does not automatically expose the others.

Three VLANs were created, each mapped to a distinct role:

| VLAN ID | Name | Purpose | Trust Level |
|---|---|---|---|
| 10 | AP | Wireless access point and its clients | Lowest — wireless/IoT-adjacent devices are the most likely initial compromise vector and should not have lateral access to anything else |
| 20 | SERVER | Homelab server (Proxmox host and its VMs/containers) | Highest sensitivity — hosts self-hosted services and should be reachable only by what explicitly needs it |
| 30 | DESKTOP_PC | Primary daily-driver workstation | Trusted — the device an administrator actually works from, and the only VLAN given a deliberate exception to reach the server VLAN |

## Implementation

All three VLANs are implemented as 802.1Q tagged sub-interfaces on a single physical interface (`igb1`) on the pfSense firewall — this is a **router-on-a-stick** design. See `04-Routing.md` for a full explanation of that design and why it was chosen over dedicating a physical NIC per VLAN.

| VLAN ID | Parent Interface | pfSense Interface Name | Subnet |
|---|---|---|---|
| 10 | igb1 | AP | 10.10.10.0/24 |
| 20 | igb1 | SERVER | 10.10.20.0/24 |
| 30 | igb1 | DESKTOP_PC | 10.10.30.0/24 |

## Current Status

The VLAN interfaces, IP addressing, and DHCP scopes are fully configured on pfSense. The Cisco Catalyst switch has not yet been configured with the corresponding 802.1Q trunk and VLAN-to-port assignments, so the physical VLAN separation is not yet live end-to-end. Until that switch configuration is complete, the access point remains connected to the untagged legacy LAN network as an interim measure. This is tracked in `07-Future-Plans.md`.

## Rule Enforcement

VLAN membership alone does not enforce security — it only creates separate broadcast domains. The actual access control between VLANs is implemented through firewall rules on pfSense, documented in `05-Firewall-Rules.md`.
