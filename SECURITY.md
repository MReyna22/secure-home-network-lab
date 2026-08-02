# Security Policy

## No deployed service

This repository contains **documentation only** — Markdown files and static
images (selectively blurred/cropped derivatives of original lab screenshots,
plus one sanitized visual summary where noted). There is no application
code, no server, no API, no container image, and no deployed service of any
kind associated with this repository. There is nothing here to scan,
exploit, or attack, and nothing here accepts network connections.

## Scope

This repository documents a personal, authorized hardening exercise performed
by the author on hardware they own. It is not software, and it does not ship
a product. Accordingly, there is no code-vulnerability disclosure process for
this repository in the traditional sense.

## If you believe you've found a genuine sensitive-data exposure

Every screenshot in this repository is a genuine original evidence image
that has had sensitive fragments (addresses, hostnames, SSIDs, credentials,
exact timestamps, fingerprint blocks) selectively blurred or cropped out and
flattened, specifically to avoid leaking real network details — see
[docs/publication-safety.md](docs/publication-safety.md) for the full
breakdown of what was redacted or generalized and why.

If you nonetheless believe you've spotted something in this repository that
could expose real, sensitive information (for example, a real IP address,
credential, or personally identifying detail that should have been
generalized), please open a GitHub issue with:

- The specific file and location
- What category of information you believe is exposed
- Why you believe it's sensitive

Please do not attempt to use any information in this repository to probe,
scan, or access any live network — none is intended to be a real, current
target, and doing so would be outside the scope of what this project
authorizes.

## Responsible disclosure boundaries

This project does not claim to have found or reported vulnerabilities in any
third party's product or infrastructure. Any vendor-side limitations
documented here (for example, in
[docs/hardening-and-validation.md](docs/hardening-and-validation.md)) reflect
publicly observable behavior of consumer firmware on hardware the author
owns, not a coordinated disclosure to the vendor. No exploitation was
attempted or is described anywhere in this repository.
