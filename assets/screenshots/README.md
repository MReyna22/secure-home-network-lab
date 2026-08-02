# Evidence Gallery — Redaction Notes

Every image in this folder is a **selectively blurred and cropped derivative of
a genuine lab screenshot** — real UI chrome, real terminal output, real scan
results — with only the sensitive fragments removed. None are recreated
illustrations or mockups. Redaction was applied as narrow, targeted blur zones
or hard crops over specific fields (addresses, hostnames, SSIDs, credentials,
exact timestamps, fingerprint blocks) — never a single opaque bar over the
whole frame, and never a reconstruction of what was hidden.

Redaction method:

- **Flattened Gaussian blur** on private/public IPs, hostnames, usernames,
  Wi-Fi profile/SSID names, MACs, firmware-version strings, and precise
  timestamps. Blur is applied as a destructive pixel operation (heavy
  downscale-blur-upscale-blur), not a reversible overlay or vector mask.
- **Hard crop** to remove entire blocks that offered no instructional value
  once the sensitive parts were considered (e.g., raw per-port fingerprint
  submission dumps).
- Where the original screenshot already had a black-bar redaction from the
  source capture, it was replaced or reinforced with the same flattened blur
  treatment for visual consistency and to close any gaps in the original bar.
- Every image was re-exported through a fresh RGB buffer, stripping all
  embedded metadata (EXIF/PNG text chunks) before being written to disk.

## Images

### `01-router-upnp-disabled.png`
Genuine NETGEAR DumaOS **UPnP** settings screen, unmodified except for the
firmware-version string in the top-right corner (blurred). Shows "Turn UPnP
On" unchecked and an empty port-mapping table — demonstrates the
service-minimization control from
[docs/hardening-and-validation.md](../../docs/hardening-and-validation.md).

### `02-guest-network-isolation.png`
Genuine Guest Network Settings screen for both the 2.4GHz and 5GHz radios.
The real guest SSID fields and the WPA3 passphrase/security-key fields are
covered with soft neutral `[REDACTED]` / `[LAB-GUEST-SSID]` masks (flattened,
not reversible); the firmware string is blurred. Everything else — the
"Enable Guest Network," "Enable SSID Broadcast," isolation checkbox, and
security-mode options — is the original, unedited interface.

### `04-router-service-enumeration-summary.png`
Genuine `nmap -sV --top-ports 20` terminal output against the lab router.
The scan command's target flag, the exact start timestamp, and the target IP
in "Nmap scan report for ..." are blurred. The port/state/service/version
table itself — the actual scan result — is fully intact and unedited.

### `06-endpoint-network-profile-baseline.png`
Genuine PowerShell output from `Get-NetConnectionProfile` and
`Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction,
DefaultOutboundAction` on the Windows endpoint. The network profile `Name`
field (which contained the real Wi-Fi profile name) and the terminal's
copyright/banner line at the top are blurred; the interface alias, network
category, connectivity state, and the firewall-profile enabled/action table
are genuine and unedited. This documents the endpoint's baseline firewall
posture (all three profiles enabled, default-deny inbound) referenced in
[docs/hardening-and-validation.md](../../docs/hardening-and-validation.md).

## What was excluded and why

Several original screenshots were **not** used at all, because no safely
redactable version of them would retain enough of the original layout to look
intentional, or because the risk of residual exposure was too high relative
to the instructional value:

- Full TLS cipher-suite enumeration and the DumaOS certificate's MD5/SHA-1/SHA-256
  fingerprints — detailed enough to help fingerprint the exact device.
- The broad port scan's raw per-port service-fingerprint submission blocks and
  a scan showing a mismatched-vendor CPE fingerprint string — verbose
  vendor-probe strings with limited instructional value relative to their
  exposure.
- Screenshots whose sensitive fields (raw MAC address tables, full certificate
  detail views) covered most of the visible frame, leaving nothing to
  usefully preserve once redacted.
- **The Kali VM bridged-network terminal transcript** (previously
  `03-vm-bridged-network-validation.png`) — the number of distinct sensitive
  fields packed into a small terminal window (hostname, LAN/Docker addresses,
  bridge interface ID) required enough separate blur blocks that the result
  read as heavily patched rather than clean, on final quality review.
- **The Windows Firewall drop-log excerpt** (previously
  `05-windows-firewall-drop-correlation.png`) — redacting every sensitive
  column (time, protocol, source IP, destination IP, source port) across
  every visible row left the majority of the log's content obscured, on
  final quality review.

Both of the above steps are still validated and documented narratively in
[docs/hardening-and-validation.md](../../docs/hardening-and-validation.md).
These findings, along with the other excluded categories above, are
described narratively, with severity and status, in
[docs/hardening-and-validation.md](../../docs/hardening-and-validation.md) and
[docs/publication-safety.md](../../docs/publication-safety.md).
