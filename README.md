# Network-Based Firewall Policy with pfSense: Aliases, Port Blocking, and Scheduled Rules

Two hands-on labs using pfSense, an open-source network firewall platform, to implement network-wide traffic policy: blocking a specific website via a reusable alias, blocking an entire insecure port across all destinations, and making a rule active only during scheduled hours.

## Scenario
Unlike host-based firewalls (one machine at a time), pfSense sits at the network level, so a single rule applies to every device on the LAN. These labs covered three policy patterns: scalable website blocking, protocol-wide port blocking, and time-scoped enforcement.

## What I did

### Exercise 1: Blocking a website via alias
- Confirmed a test site loaded normally from an admin machine before any rule existed
- Resolved the site's IP with `ping rediff.com`, noting that a domain can resolve to multiple, changing IPs
- Created a pfSense **alias** (Firewall → Aliases) named `BlockedWebsites` (type: Host(s)), containing both the domain and its resolved IP
- Created a firewall rule (Firewall → Rules → LAN): Action = Block, Protocol = TCP/UDP, Source = any, Destination = Single host or alias → `BlockedWebsites`, port range = any
- Verified from a different machine on the network that the site was now unreachable ("This site can't be reached") — confirming network-wide enforcement, not just a single-host block

### Exercise 2: Blocking an insecure port (HTTP/80)
- Confirmed a target site loaded over plain HTTP before any rule
- Created a rule: Action = **Reject**, Protocol = TCP/UDP, Source = any, Destination = any, Destination Port Range = HTTP (80)
- Verified HTTP access failed while HTTPS to the same site remained reachable — the rule was scoped to the insecure port specifically, not the site as a whole
- Deleted the rules from both exercises and rebooted pfSense to restore a clean baseline, confirmed by re-testing site access

### Exercise 3: Time-scoped enforcement with Schedules
- Created a pfSense **Schedule** (Firewall → Schedules) named `WorkingHours`, with a specific day and a start/stop time (noting schedules can't cross midnight — a full day is 0:00–23:59)
- Created a new HTTP-blocking rule, then under Advanced Options attached the `WorkingHours` schedule — making the rule active only during that defined window instead of permanently
- Verified the rule enforced correctly within the scheduled window
- Cleaned up by deleting the rule and rebooting pfSense again

## Key takeaways
- **Aliases decouple policy from infrastructure**: a rule referencing an alias survives IP changes and list growth without ever being edited itself — a rule hardcoded to one IP breaks the moment that IP changes
- **Port-based rules and host-based rules scale in different directions**: blocking a port affects every destination at once; blocking a host affects one destination across every port — choose based on what's actually being prevented
- **Reject vs. Block is a deliberate choice, not interchangeable**: Reject sends an active response (fast, clear failure — good for enforcing policy on your own users); Block silently drops packets (gives an adversary less information — better for more adversarial contexts)
- **Schedules let policy reflect how organizations actually operate** — very few rules should be static 24/7, and attaching a schedule avoids manually toggling rules on a clock
- Always verify enforcement from a different machine on the network, and restore a clean baseline (delete test rules, reboot) before moving on

## Tools
pfSense (Firewall Aliases, Rules, Schedules), Google Chrome, Windows Command Prompt (`ping`)

---
*Completed in an isolated training lab environment against a provisioned pfSense firewall and network of VMs.*
