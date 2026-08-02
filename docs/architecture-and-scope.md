# Architecture and Scope

## What this project is

A hands-on hardening and validation exercise performed on a personally owned
consumer router (NETGEAR Nighthawk XR1000 v1, running DumaOS), staged behind
an existing ISP-provided gateway. The goal was to practice a disciplined,
change-managed approach to securing real hardware: establish a baseline,
make deliberate changes, validate the outcome with actual tests, investigate
anything unexpected, and document limitations honestly rather than paper over
them.

## What this project is not

- **Not enterprise infrastructure.** There is no VLAN segmentation, no
  centralized logging or SIEM, no certificate authority, no IDS/IPS, and no
  formal change-approval process. The router is consumer-grade hardware with
  consumer-grade ceilings on what can be configured.
- **Not a professional penetration test.** Scanning was limited to
  authorized, personally owned equipment on the trusted LAN. No exploitation,
  no attempt to bypass authentication, and no Internet-side exposure testing
  was performed.
- **Not employment or contracted work.** This is an independent portfolio
  project intended to build and demonstrate practical skill, not a record of
  paid professional engagement.

## Environment

| Component | Role |
|---|---|
| ISP-provided gateway | Upstream router, out of scope for hardening — kept in place so the household's primary connectivity was never put at risk |
| Lab router (NETGEAR XR1000 v1 / DumaOS) | Primary hardening and test subject |
| Windows workstation | Management endpoint used to administer the lab router and later reviewed itself for exposure |
| VMware Workstation + Kali Linux VM | Authorized scanning and validation host |
| Mobile device | Secondary client used for trusted and guest wireless testing |

The lab router was deployed **behind** the existing household gateway rather
than replacing it. This double-NAT staging design was a deliberate choice: it
let the project experiment freely — including a factory reset and firmware
changes — without touching the household's primary Internet connectivity or
putting other household users at risk of an outage.

## Tooling

- **Nmap** — host discovery, service enumeration, and TLS/cipher review, run
  only against authorized lab hosts from the trusted LAN
- **PowerShell** (`Get-NetFirewallRule`, `Get-NetTCPConnection`,
  `Get-Service`, `Get-AuthenticodeSignature`) — Windows-side service,
  listener, and code-signature review
- **Windows Defender Firewall logging** — drop-log correlation against the
  Nmap scans
- **curl** — direct HTTP/HTTPS behavior verification against the router's
  admin interface
- **VMware Workstation** — network mode comparison (NAT vs. Bridged) for the
  Kali Linux test VM

## Constraints

- All scanning was performed **from the trusted LAN only.** No Internet-side
  exposure test was completed, and this project does not claim that any
  service documented here is reachable from the public Internet.
- The lab router's firmware, feature set, and available hardening controls
  are entirely vendor-defined (DumaOS). Several findings in this project
  exist specifically *because* the interface offered no control to address
  them — see [hardening-and-validation.md](hardening-and-validation.md) and
  [publication-safety.md](publication-safety.md).
- The original initial-setup steps were not captured contemporaneously with
  screenshots; any later recreation of that sequence is documented as
  reconstructed process narrative, not as originally captured evidence.

## What this design does not claim

- It does not claim network segmentation beyond the router's built-in guest
  SSID isolation.
- It does not claim the router is free of vendor-side vulnerabilities —
  several were identified and are documented as residual risk rather than
  resolved.
- It does not claim exhaustive service identification. Nmap's own
  service-fingerprint guesses were inconsistent for a handful of high-numbered
  ports and are recorded as an open, unconfirmed observation rather than a
  finding.
- It does not claim this configuration is permanent or maintenance-free —
  ongoing verification (e.g., of firmware currency) is called out explicitly
  as future work.
