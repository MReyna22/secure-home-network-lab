# SIM-NET-001: Local management page unavailable while VPN is active

> **Simulated training ticket.** Based on a documented lab troubleshooting lesson. This is not a production ticket, a new live test, or proof of a measured SLA. Device/vendor names, network addresses, account details, and historical timing are omitted.

## Intake

| Field | Training record |
|---|---|
| Environment | Generic workstation, local management page, VPN client |
| Symptom | Local management page fails while the VPN is active |
| Impact | Management task interrupted; scope of any other outage is unknown |
| Priority | P3, an illustrative training choice |
| Status | Closed for the example after access returns |
| Timing | No elapsed time or incident date asserted |

## Suggested questions

1. What exact error appears, and what changed?
2. Is only this local page affected?
3. Is a VPN active, and is local access permitted by policy?
4. Can another authorized device reach the page?
5. Is this personally controlled lab equipment or an employer-managed endpoint?

These are example intake questions, not a record of historical customer answers.

## Diagnosis, fix, and verification

The documented lesson is that a VPN can interfere with local management access. In the source lab notes, disconnecting the VPN restored the management task. The exact route or policy mechanism was not independently established.

For this example, the approved comparison is to check access with the VPN state changed **only when authorized in that environment**. Verify the original management task, record the observed result, and stop short of calling it a permanent policy fix.

No packet capture, repeatability result, restart test, or resolution-time metric is invented.

## Example closure note

“Local management access is restored in this lab example. VPN state was associated with the failure. A permanent, policy-compliant configuration fix still requires investigation.”

## Escalation judgment

For an employer-managed endpoint, follow the approved support workflow. Do not disable a required VPN or weaken security controls as a shortcut. Escalate the symptom, approved test results, relevant change history, and client state to the responsible team.

## Follow-up evidence

- [ ] Sanitized before/after test
- [ ] Approved local-access policy and client version
- [ ] Controlled reproduction on authorized equipment
- [ ] Original task verification
- [ ] Restart/reconnection behavior and remaining limits

These are planned evidence items, not completed tests.

**Demonstrates:** scoping, diagnostic reasoning, verification, documentation, user communication, and escalation.
