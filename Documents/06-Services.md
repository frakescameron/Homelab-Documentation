# Services

This document covers the services and packages configured directly on pfSense that support the rest of the network: DNS, DHCP, remote administration, intrusion detection, and content filtering.

## DNS Resolver (Unbound)

pfSense's built-in DNS Resolver (Unbound) handles all DNS resolution for every VLAN. Two hardening measures were added on top of the default configuration:

- **DNSSEC validation** — enabled so that Unbound cryptographically verifies DNS responses against a domain's published signature before trusting them, protecting against cache poisoning and forged DNS answers.
- **DNS-over-TLS (DoT) to upstream** — configured via a custom forwarding zone so that Unbound's outbound queries to Cloudflare (1.1.1.1 / 1.0.0.1) are encrypted in transit, rather than sent as plaintext where they could be observed or tampered with on the path out to the internet.

These two protections solve different problems: DNSSEC verifies that an answer is authentic, while DoT ensures that a query and its answer can't be read or altered by anything sitting between pfSense and the upstream resolver. Together, DNS lookups on this network are both authenticated and encrypted end-to-end from the client to the upstream provider.

## DHCP (Kea)

DHCP is served by pfSense's **Kea** backend rather than the legacy **ISC DHCP** backend. ISC DHCP has reached end-of-life within pfSense and is scheduled for eventual removal from future releases, so the migration to Kea was done proactively rather than waiting to be forced into it.

**Known limitation:** as of the current pfSense release, Kea does not support automatically registering DHCP leases or static mappings into the DNS Resolver the way ISC DHCP did. This means devices are not currently resolvable by hostname through internal DNS purely from their DHCP lease. This is a known, tracked gap in Kea's current feature set rather than a misconfiguration, and is expected to be addressed in a future pfSense release.

## SSH Administration

Remote administration of pfSense over SSH is restricted to **key-based authentication only**; password authentication for SSH has been disabled. An ed25519 keypair was generated on the administrator's workstation, and the public key was added to the pfSense admin account before password login was turned off, with the key tested successfully prior to disabling the password fallback. The pfSense web administration interface and SSH are both unreachable from WAN (see `05-Firewall-Rules.md`), so administrative access requires being on the internal network in addition to holding the private key.

## Suricata (Intrusion Detection)

Suricata is installed and running on the WAN interface in **IDS mode (detection only, not blocking)** using the free ET Open (Emerging Threats) ruleset. Hardware checksum/TSO/LRO offloading was disabled network-wide, which Suricata requires in order to inspect packets correctly.

The plan is to run in detection-only mode for a review period (roughly a week of normal daily usage) to identify any rules that false-positive against legitimate traffic, suppress those specific signatures, and only then switch to IPS (blocking) mode. This order matters — enabling blocking before validating the ruleset against real traffic risks silently dropping legitimate connections with no easy way to tell why something suddenly stopped working.

## pfBlockerNG (DNSBL)

pfBlockerNG-devel is installed and configured for **DNSBL** (DNS-based blocklisting) in Unbound mode, integrating directly with the DNS Resolver above. It blocks DNS resolution for domains on the StevenBlack ad/tracker/malware blocklist, serving a block page from an isolated virtual IP (172.16.0.1 — see `02-Ip-Addressing.md` for why that specific address was chosen) when a device requests a blocklisted domain.

This provides network-wide ad and malware-domain blocking for every device automatically, with no per-device configuration required, functionally similar to a dedicated Pi-hole instance but implemented at the firewall layer. GeoIP-based blocking (blocking traffic by country) is supported by pfBlockerNG but is not currently enabled.

## Summary Table

| Service | Package/Feature | Status |
|---|---|---|
| DNS Resolver | Unbound + DNSSEC + DNS-over-TLS | Complete |
| DHCP | Kea DHCP | Complete |
| SSH Administration | Key-only authentication | Complete |
| Intrusion Detection | Suricata (ET Open, IDS mode) | Running, under evaluation before enabling blocking |
| DNS-based Ad/Malware Blocking | pfBlockerNG-devel (DNSBL) | Complete |
