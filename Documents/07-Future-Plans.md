# Future Plans

This document tracks planned improvements to the network, roughly in the order they are expected to be tackled.

## Near-Term

- **Complete the switch trunk configuration** — configure the Cisco Catalyst switch with an 802.1Q trunk to pfSense carrying VLANs 10/20/30, and configure access ports for the AP, Proxmox host, and desktop PC on their respective VLANs. This is the last step needed to make the VLAN design fully live end-to-end (currently the AP is temporarily connected to the untagged legacy LAN).
- **Enable Suricata IPS (blocking) mode** — after a review period of IDS-only logging to confirm no legitimate traffic is being false-positived, switch Suricata from detection-only to actively blocking malicious traffic on WAN.
- **Enable logging on firewall block rules** — currently the inter-VLAN block rules drop traffic silently; enabling logging on them will make denied cross-VLAN traffic visible in the firewall logs for review and troubleshooting.
- **Automated configuration backups** — configure pfSense's Auto Config Backup and/or a self-hosted scheduled backup of the running configuration to the Proxmox server, so the firewall configuration isn't a single point of failure.

## Medium-Term

- **WireGuard VPN** — for secure remote access into the homelab without exposing any service or the pfSense admin interface directly to the internet.
- **Config-change notifications** — email or messaging alert whenever the pfSense configuration changes, to catch unauthorized or unintended changes.
- **Traffic visibility (ntopng)** — deeper flow-level traffic visibility as a complement to Suricata's threat detection.
- **Port security, DHCP Snooping, and Dynamic ARP Inspection** on the Cisco switch, along with EtherChannel (LACP), as additional CCNA-relevant switching features to implement hands-on.

## Longer-Term

- **Active Directory Domain Services, Windows DNS/DHCP, and Group Policy** on the Windows Server VM, with the Windows client VM domain-joined. Once this is live, this documentation will be updated to clarify which system (pfSense or the domain controller) is authoritative for DNS/DHCP to avoid any conflicts between the two.
- **IPv6** — currently the network is IPv4-only; IPv6 addressing and firewall rules are a planned future project.
- **High Availability (HA)** — a second pfSense node in a CARP failover pair. More of an "enterprise resume" exercise than a practical necessity at this scale, but valuable to have hands-on experience with.
- **DOCSIS 3.1 modem upgrade** — the current Motorola MB7621 (DOCSIS 3.0) is the primary bottleneck against the 1 Gbps internet plan.
- **Additional Linux servers** (RHEL, Arch, Debian) on the Proxmox host for broader hands-on systems administration practice.

## Notes

This list will be updated as items are completed and new priorities emerge. Completed items are moved into their relevant document (e.g. `05-Firewall-Rules.md`, `06-Services.md`) rather than staying listed here.
