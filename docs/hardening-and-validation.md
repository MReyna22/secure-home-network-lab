# Hardening and Validation

Every control below follows the same pattern: **initial state → why it
mattered → what was done → how it was validated → outcome.** A setting being
enabled or disabled is treated as configuration evidence, not proof — controls
were only accepted once tested.

---

## Firmware currency

**Initial state:** The router's built-in update checker reported "no new
firmware version available" while running an older build.

**Why it mattered:** A router's own updater is not a trustworthy sole source
of patch status — if the mechanism itself is stale or misreporting, unpatched
vulnerabilities can sit undetected indefinitely.

**What was done:** Independently checked the vendor's official support page
for the exact hardware revision, confirmed a newer package existed, exported
a configuration backup as a rollback point, downloaded and manually applied
the correct firmware image.

**Validation:** After reboot, logged back in and reviewed every major
configuration section (WAN/DHCP, LAN subnet, wireless settings, guest
isolation, UPnP state, administrator credentials, automatic-update settings)
to confirm nothing had silently reset during the update.

**Outcome:** Remediated. Firmware confirmed current against the vendor's own
published version, with no unintended configuration drift.

> **Change-management lesson:** treat a router firmware update like any other
> controlled change — verify hardware revision, back up configuration, apply
> the vendor package, confirm the new version, and validate service
> continuity afterward. Don't just trust "you're up to date."

---

## Router service minimization

**Initial state and decisions:**

| Control | Initial state | Decision |
|---|---|---|
| Port triggering | Enabled, empty rule table | Disabled — no operational need, and the feature can dynamically permit inbound traffic |
| DLNA media server | Enabled | Disabled — unnecessary service exposure |
| ReadySHARE Network Neighborhood / MacShare | Enabled | Disabled — USB file sharing not required |
| ReadySHARE local HTTP | Enabled | Disabled — USB file sharing not required |
| UPnP | Disabled | Retained disabled — no automatic port-map entries were present |
| Dynamic DNS | Disabled | Retained disabled |
| VPN server | Disabled | Retained disabled — remote VPN access out of scope |
| IPv6 | Disabled | Retained disabled during the IPv4-focused phase (not treated as inherently insecure) |
| Automatic firmware updates | Enabled | Retained, with manual vendor verification still treated as required |

**Validation:** Confirmed via the router admin interface after each change
that the corresponding feature state and (for UPnP) the port-mapping table
matched expectations.

**Outcome:** Completed. Evidence: [UPnP disabled with an empty mapping table](../assets/screenshots/01-router-upnp-disabled.png).

**WPS status (unresolved item, documented honestly):** the WPS wizard itself
was available, but a separate control to disable the router PIN specifically
could not be located in the interface. No active PIN was displayed. This
project does **not** claim WPS was fully disabled — that status is recorded
as not independently verified rather than assumed safe.

---

## Guest network isolation

**Why it mattered:** A guest network is only a meaningful control if guests
actually can't reach anything but the Internet. A checked "enable isolation"
box is a configuration claim, not a security guarantee.

**What was done:** Configured separate 2.4 GHz and 5 GHz guest SSIDs with
guest-to-LAN visibility disabled, then tested actual behavior instead of
trusting the setting.

**Validation — functional tests, not configuration review:**

| Test | Observed result |
|---|---|
| Trusted 2.4 GHz connectivity | Passed on phone and computer |
| Trusted 5 GHz connectivity | Passed on phone and computer |
| Guest Internet access | Allowed |
| Guest access to trusted host | Blocked |
| Guest access to router administration | Blocked |
| Guest-to-guest access | Blocked |

**Outcome:** Validated. Evidence: [guest network isolation configuration](../assets/screenshots/02-guest-network-isolation.png).

> **Control-validation principle applied throughout this project:** a checked
> box is treated as configuration evidence, not proof. A control is only
> accepted once it's been tested to actually behave as intended.

---

## VMware bridged-mode migration and asset discovery

**Initial state:** The Kali Linux VM ran in VMware NAT mode, placing it behind
a separate private address space rather than as a real peer on the lab LAN.
This limited its usefulness as an independent test host — from the router's
perspective, the VM didn't really exist as its own device.

**What was done:** Switched the VM's network adapter from NAT to Bridged
mode so it received an address directly from the lab router and behaved as a
distinct LAN-side client.

**Validation:** Confirmed the new bridged address, gateway, and DNS settings,
then tested gateway reachability, public Internet reachability, and DNS
resolution — all passed. A follow-up host-discovery scan confirmed four
distinct active lab devices.

**A limitation worth documenting:** the router's own device list associated
the VM's address with the Windows host's physical Wi-Fi identity — a known
quirk of bridging a VM through a wireless adapter. Rather than trusting a
single tool, multiple sources (VM adapter state, host-discovery scan results,
and the router's device list) were reconciled to build an accurate picture of
which address belonged to which device.

**Outcome:** Validated. (Supporting terminal evidence for this step was excluded from the published screenshot set — see [assets/screenshots/README.md](../assets/screenshots/README.md#what-was-excluded-and-why).)

---

## Router management and service assessment

**What was done:** Ran an authorized, LAN-scoped Nmap service scan against
the lab router, limited to its top 20 ports.

**Result (generalized):** DNS, HTTP, and HTTPS were open and expected for a
router admin interface. One additional high-numbered port returned an
unrecognized service banner and was logged for follow-up rather than assumed
safe. Common attack-surface services — FTP, SSH, Telnet, SMB, RDP, VNC, and
common database ports — were all closed.

**HTTPS enforcement behavior:** Enabling "always use HTTPS" surfaced a
browser-compatibility limitation in the router's own interface, which
required switching the administrative browser. A later access failure turned
out to be caused by VPN client behavior, not the router — see
[troubleshooting-and-lessons.md](troubleshooting-and-lessons.md).

**Independent verification:** Direct HTTP/HTTPS requests against the router
confirmed both ports required authentication, but also confirmed **HTTP
remained reachable** even with the HTTPS preference enabled. This is recorded
as a documented residual risk, not resolved by the interface setting alone.
Operational mitigation: administer only through an HTTPS session, from the
trusted LAN, and never submit credentials over HTTP.

**Certificate and TLS limitations (vendor-side, unresolved):** the router's
local management certificate had already expired, and the interface offered
TLS 1.0 and 1.1 alongside 1.2 with no control to disable the deprecated
versions. Detailed cipher lists and certificate fingerprints are intentionally
not published here — see [publication-safety.md](publication-safety.md) for
why.

**Outcome:** Mitigated / residual. Evidence: [router service enumeration, generalized](../assets/screenshots/04-router-service-enumeration-summary.png).

---

## Windows endpoint exposure validation

**Why it mattered:** A secure network isn't just the router — the managed
client sitting on it matters just as much.

**What was done:** Classified the router connection as a Public network in
Windows, confirmed Windows Defender Firewall was enabled across all profiles,
and kept the stricter Public profile since inbound sharing wasn't needed.

**Validation:** PowerShell showed several local listeners on the Windows
host, but an authorized external top-100 TCP scan from the Kali VM reported
every one of those ports as filtered — direct proof that a local listener is
not automatically a network-reachable one. Windows Firewall drop-packet
logging was then enabled, and the resulting log entries lined up exactly with
the ports probed during the scan.

**Outcome:** Validated. (Supporting firewall-log evidence for this step was excluded from the published screenshot set — see [assets/screenshots/README.md](../assets/screenshots/README.md#what-was-excluded-and-why).)

---

## Endpoint findings and remediation

### Windows Delivery Optimization (peer-to-peer update sharing)

**Finding:** A targeted scan found an open port mapping to Windows Delivery
Optimization, with its inbound TCP/UDP firewall rules enabled for all
profiles — while telemetry showed no actual peer transfers or benefit in this
environment.

**Remediation:** Disabled "Allow downloads from other PCs" without disabling
Windows Update itself. The local listener disappeared, and a retest from the
Kali VM reported the port filtered.

**Outcome:** Remediated.

### Two bundled third-party services (Wondershare Dr.Fone components)

**Finding:** PowerShell identified two related services running automatically
as `LocalSystem`. One executable had a valid digital signature; the other was
unsigned. Both were already blocked from the LAN by Windows Firewall, but
they still increased the local privileged-service footprint unnecessarily.

**Remediation:** Both services were stopped and removed from active
execution. Follow-up checks confirmed no related process and no listener
remained on either associated port, and a Kali retest continued to report
both as filtered.

**Outcome:** Remediated in current state. A reboot-persistence check (do the
services re-register themselves after a restart) remains a recommended
follow-up rather than a confirmed closed item.

Evidence: [endpoint network profile and firewall-profile baseline](../assets/screenshots/06-endpoint-network-profile-baseline.png) — confirms all three Windows Firewall profiles enabled with default-deny inbound on the endpoint where these services were found and remediated.

---

## Final router scan and residual risk

A broader, final authorized TCP scan of the router (still LAN-scoped) found
the expected DNS/management services plus several additional high-numbered
listeners. Nmap's own service-identification guesses for those ports were
inconsistent — one was even fingerprinted as belonging to an entirely
different vendor's product, despite being run against this router. That
result is recorded as an **open observation requiring vendor-specific
interpretation**, not as proof of a separate device or a confirmed
vulnerability.

> **Scope limitation:** every scan in this project was performed from the
> trusted LAN. No Internet-side exposure test was completed. This project does
> not claim that any listed management or internal service is reachable from
> the public Internet.

## Findings summary

| ID | Finding | Severity | Status | Disposition |
|---|---|---|---|---|
| F-01 | Outdated router firmware | High | Remediated | Manual vendor update completed and validated |
| F-02 | Port triggering enabled without a requirement | Low–Medium | Remediated | Feature disabled |
| F-03 | DLNA media server enabled without a requirement | Medium | Remediated | Service disabled |
| F-04 | Unused ReadySHARE local services enabled | Medium | Remediated | Network Neighborhood/MacShare and local HTTP disabled |
| F-05 | HTTP management remained reachable after HTTPS preference was enabled | Medium | Mitigated / residual | Firefox over HTTPS only, from trusted LAN; guest access blocked |
| F-06 | Expired local management certificate | Low–Medium | Residual vendor limitation | Session still encrypted, but browser identity validation is weakened |
| F-07 | TLS 1.0 and 1.1 still supported | Medium | Residual vendor limitation | No interface control found; management restricted to trusted LAN |
| F-08 | Additional unidentified vendor-managed TCP services | Informational | Open observation | Documented; no vulnerability claimed, no WAN test performed |
| F-09 | Windows Delivery Optimization peer service exposed | Low–Medium | Remediated | Peer downloads disabled; listener removed; retest filtered |
| F-10 | Privileged third-party services running unnecessarily | Medium | Remediated in current state | Services stopped; reboot-persistence check recommended |
