# Publication Safety

This document explains, category by category, what was removed or generalized
before publishing this repository, and why. The intent is secure documentation
hygiene, not evasiveness — every decision below trades a small amount of
specificity for a meaningful reduction in real-world exposure, without
weakening the technical narrative.

## Source material handling

The original project report (a Word document) is used only as private
reference material while building this repository and is **not included
here**. Most images in [assets/screenshots/](../assets/screenshots/) are
selectively blurred/cropped derivatives of the original evidence
screenshots: real UI chrome and real terminal/tool output, with sensitive
fields removed via narrow, flattened blur zones or hard crops rather than a
full-frame black bar. This differs from a naive redaction that only covers
the visible layer of an existing image (for example, a thin black bar with
gaps) — every redaction here is a destructive pixel operation applied
directly to the image and re-exported through a fresh RGB buffer, so no
underlying pixel data survives and no embedded file metadata (capture
device, timestamps, software identifiers) is carried over. Where no version
of an original screenshot could be made both safe and useful, it was left
out entirely rather than replaced with an invented mockup; see
[assets/screenshots/README.md](../assets/screenshots/README.md) for the
per-image treatment.

## Categories redacted or generalized, and why

| Category | Treatment | Why |
|---|---|---|
| Private LAN/subnet addresses (including the router's LAN range and any VMware NAT range) | Replaced with `[LAB-GATEWAY]`, `[LAB-CLIENT-IP]`, `[LAB-SUBNET]`, `[LAB-SCANNER-IP]` | Real addressing has no instructional value on its own and unnecessarily ties published material to one specific household network |
| WAN/public IP addresses | Never shown, not even redacted-in-place | A public IP is a direct identifier of the household's Internet connection |
| Router access URLs / local management hostnames | Not shown | Combined with other details, these can help locate or fingerprint the specific device |
| Wi-Fi network names (SSIDs) | Replaced with generic placeholders (`[LAB-GUEST-SSID]`) | Real SSIDs are often personally chosen and can be identifying or searchable |
| Wi-Fi passwords, router admin credentials, recovery answers | Never shown | Direct credential exposure |
| Router serial numbers, PINs, and hardware IDs | Not shown | Can be used for warranty/support-channel impersonation or device-specific lookups |
| Full MAC addresses | Not shown | Persistent hardware identifiers |
| Hostnames, shell usernames, and device names (e.g., a personal machine or VM hostname visible in a terminal prompt) | Not shown; any terminal evidence was rebuilt with generic prompts | Personally chosen device/user names are identifying and often reused across other accounts |
| Browser profile information | Not shown | Not relevant to the technical findings and potentially identifying |
| Certificate fingerprints (MD5/SHA-1/SHA-256) and full TLS cipher enumeration | Generalized to a narrative finding ("TLS 1.0/1.1 still supported; certificate expired") rather than the raw values | Detailed fingerprints and cipher lists can help profile or target the exact device; the finding itself is what matters for the portfolio, not the raw values |
| Exact scan timestamps | Generalized to `[date]` or omitted | Precise timestamps add no instructional value and can support correlation/profiling when combined with other data |
| Raw Nmap service-fingerprint submission blocks | Omitted; replaced with a plain-language summary of open/closed state | These are verbose vendor-probe strings intended for Nmap's own database, not meaningful evidence for a reader, and they can reveal granular version data useful for targeting the device |
| Configuration backup files | Never included | Vendor configuration exports can bundle several of the categories above into one file |
| Docker/bridge interface names, VM internal identifiers | Not shown | Incidental identifiers with no bearing on the findings |

## Screenshots that were excluded entirely

Some original evidence was not recreated or referenced at all, because no
sanitized version could preserve enough instructional value to be worth
publishing:

- The full TLS cipher-suite enumeration output and the router's certificate
  fingerprint details (see table above).
- The raw, multi-port service-fingerprint submission dump from the final
  broad scan.
- Any screen where a real SSID or Wi-Fi password-hint text was still visible
  alongside an existing black-bar redaction in the source material — the
  project's own working assumption is that a redaction covering *most* of a
  sensitive field is not the same as removing it, so those were left out
  rather than partially trusted.

The corresponding findings from all of the above are still documented — in
plain language, with severity and status — in
[hardening-and-validation.md](hardening-and-validation.md). Excluding the
image never means excluding the finding.

## What was intentionally *not* generalized

A few details were kept specific because they carry real instructional value
and don't meaningfully increase risk:

- Router hardware model (NETGEAR Nighthawk XR1000 v1) and firmware version
  numbers — mentioned in high-level context only, consistent with the goal of
  a credible, honest portfolio artifact rather than an anonymous exercise.
- Public, non-identifying reference values, like the well-known public DNS
  test address `1.1.1.1` used for connectivity checks.
- Named third-party software components identified during endpoint review
  (e.g., specific service display names) — these are public product/vendor
  identifiers already documented by the software vendors themselves, not
  personal or network-identifying data.

## Ongoing publishing caveats

- This repository does not claim to have identified every sensitive detail
  that could theoretically exist in a home-network project. The review
  focused on the categories above because they map directly to realistic
  risks (device profiling, credential exposure, personal identification).
- If this lab is expanded in the future (see the README's future-expansion
  section), any new evidence should go through the same review before
  publication: rebuild rather than crop, generalize addressing and
  timestamps, and exclude anything that only makes sense with a real device
  in hand.
