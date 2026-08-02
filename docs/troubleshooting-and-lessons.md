# Troubleshooting and Lessons

Several parts of this project didn't go as expected the first time. Each one
turned into a small troubleshooting exercise, and each taught something worth
recording — including cases where the root cause wasn't proven, just observed
and fixed.

---

## VMware NAT vs. Bridged networking

**Symptom:** The Kali Linux VM had network connectivity under VMware's NAT
mode, but it wasn't really *visible* on the lab LAN — from the router's point
of view, it didn't exist as an independent device.

**Why it happened:** NAT mode puts the VM behind VMware's own internal
private address space and translates traffic through the host, similar to how
a second router would. That's fine for basic Internet access, but it hides
the VM as a distinct network endpoint — which mattered for a project that
specifically needed an independent LAN-side vantage point to scan from.

**Fix:** Switched the VM's adapter to Bridged mode, which put it directly on
the lab router's network and gave it its own address from the same DHCP pool
as every other device.

**Lesson:** VMware NAT and Bridged modes create genuinely different visibility
and addressing behavior — not just a cosmetic difference. Choosing the wrong
one can silently limit what a lab is actually capable of testing.

**A related wrinkle:** once bridged, the router's own device list associated
the VM's address with the physical Wi-Fi adapter identity of the host it was
running on. That's a known quirk of bridging a VM through a wireless
interface — the router sees the traffic's physical origin, not just the VM's
logical identity. Rather than trust the router's device list alone, this
project reconciled it against the VM's own reported adapter state and a
separate host-discovery scan to confirm which address really belonged to
which device.

---

## Browser/local access behavior

**Symptom:** The router's admin interface loaded fine from a phone but failed
to load from the Windows workstation's browser.

**Investigation:** Checked Windows Defender Firewall (not the cause — traffic
wasn't being blocked at the host) and then reviewed browser-level site
behavior. A Chrome site permission controlling pop-ups and redirects turned
out to be the relevant setting. Changing it restored access.

**Why this is reported carefully:** the root cause was never independently
reproduced — it's entirely possible something else changed at the same time.
This project records it as *the observed fix*, not as a proven root cause,
and that distinction matters. Don't overstate confidence in a fix just
because the symptom went away.

**Lesson:** when "the network" seems broken from only one device, check
browser-level state (site permissions, cached redirects, extensions) before
assuming it's a routing, firewall, or server-side problem.

---

## VPN interference with local management

**Symptom:** After enabling HTTPS-only management on the router, admin access
started failing intermittently.

**Investigation:** Traced the failure to ProtonVPN — the VPN client was
blocking or rerouting local-network traffic, including access to the router's
own LAN-side management address, as part of its normal behavior of directing
traffic through the VPN tunnel.

**Fix:** Disconnecting the VPN restored router management access immediately.

**Lesson:** VPN clients can interfere with local network management in ways
that have nothing to do with the target device's configuration. When
troubleshooting local access issues, endpoint networking state — including
whether a VPN is active — needs to be checked early, not as an afterthought.

---

## Service-identification caution (Nmap results are a starting point, not a verdict)

**What happened:** A broader authorized scan of the router returned several
high-numbered open ports. Nmap's own service-fingerprint guesses for some of
them were inconsistent with each other, and one port was even fingerprinted
as belonging to a completely different vendor's hardware than the device
actually being scanned.

**Why this matters:** it would have been easy to report that mismatched
fingerprint as a finding — "this device appears to be running another
vendor's firmware component" — but that would have been an overclaim. Service
identification from a scanner is *evidence to investigate*, not automatic
proof of what's actually running. This project records that result as an
open, unconfirmed observation requiring vendor-specific interpretation, not as
a vulnerability or a misidentified device.

**Lesson:** treat automated service/version identification as a lead, not a
conclusion — especially on consumer hardware where vendors sometimes reuse
or license components from other manufacturers.

---

## Local listener vs. network-reachable port

**What happened:** PowerShell showed the Windows management endpoint had
multiple ports in a listening state locally. A separate, authorized external
scan from the Kali VM reported all of those same ports as filtered — not
open, not closed, but silently dropped.

**Why this matters:** a listening service and a network-reachable service are
two different questions answered by two different tools. Host-level
enumeration tells you what's running; network-level scanning tells you what's
actually exposed *through* the firewall. Confusing the two can lead to either
false alarm (assuming a listener is automatically dangerous) or false comfort
(assuming a closed network port means nothing is running).

**Lesson:** correlate host-level and network-level evidence before drawing a
conclusion about exposure. This project did that explicitly by matching the
scan's probed ports against the Windows Firewall's own drop log line-for-line.
