# P1 Incident Response and Escalation (NetApp ONTAP)

## Purpose and scope

How L1/L2 storage operations respond to a Priority 1 (P1) storage incident on NetApp FAS/AFF systems: first response, evidence capture, escalation to NetApp support and the Support Account Manager (SAM), stakeholder updates and the post-incident review.

## Severity definitions

| Priority | Definition | Example | Response target |
|---|---|---|---|
| P1 | Production data unavailable, or at risk of loss, for many users or a critical service | Aggregate offline, HA pair down, datastore inaccessible to VMware cluster | Engage within 15 min, NetApp case P1 by phone |
| P2 | Service degraded, redundancy lost, no outage yet | One node in takeover, one SAN fabric down, SnapMirror broken | Engage within 1 h |
| P3 | Minor fault, no user impact | Single failed disk with spare rebuilding, fan alert | Next business day |

When in doubt, raise as P1 and downgrade later.

## Roles

- **Incident lead:** owns the timeline and decisions until handover.
- **Storage engineer:** runs diagnostics and fixes.
- **Communications:** sends stakeholder updates (can be the incident lead on a small team).
- **NetApp support / SAM:** vendor technical support; the SAM escalates inside NetApp when a case stalls.

## First 15 minutes

1. **Confirm the alert is real.** Check from more than one source (Unified Manager / Data Infrastructure Insights alert, host-side errors, user reports).
2. **Scope the impact.** Which cluster, node, aggregate, SVM, volumes or LUNs? Which applications and hosts?
3. **Capture evidence before changing anything:**
   ```
   system health status show
   system health alert show
   storage failover show
   storage aggregate show -fields state,percent-used
   volume show -state !online
   network interface show -status-oper down
   event log show -severity ERROR,EMERGENCY -time >1h
   ```
4. **Open the incident** in the ticketing system with the scope, start time and evidence.
5. **Send the first notification** (template below).

## Escalate to NetApp

1. Trigger an AutoSupport so NetApp has current data:
   ```
   system node autosupport invoke -node * -type all -message "P1 <ticket-id>"
   ```
2. Open a P1 case **by phone** (a web case alone is too slow for P1). Have ready: system serial numbers, ONTAP version, ticket ID, impact statement, timeline, evidence above.
3. Record the case number in the ticket.
4. **Escalate to the SAM** if:
   - no engineer has engaged within the agreed response time,
   - the case needs a different skill group or a hardware dispatch is delayed,
   - the incident is approaching the business recovery deadline.

## Hardware replacement

- If NetApp dispatches a part (disk, controller module, PSU), confirm the delivery time and who receives it on site.
- Before replacing, confirm the failed component's serial and slot against the case.
- Follow the NetApp replacement procedure for that part; do not improvise.
- After replacement: re-run the health checks above and confirm failover state is normal (`storage failover show`).

## Stakeholder updates

- First notice within 15 minutes, then **every 30 minutes** until resolved, even if there is no change.

Template:

> **[P1] <system> – <one-line impact>**
> Status: Investigating / Mitigated / Resolved
> Impact: <services, users>
> Started: <time>
> Current action: <what is being done now>
> NetApp case: <number>
> Next update: <time>

## Recovery and close

- Confirm with application owners that service is restored, not only that the storage is healthy.
- Leave the NetApp case open until the root cause is confirmed.
- Hand over to the next shift with the ticket fully updated if the incident runs past shift end.

## Post-incident review (within 5 working days)

1. Timeline (detection, escalation, mitigation, resolution)
2. Root cause (from NetApp's analysis where relevant)
3. What went well / what slowed us down
4. Actions with owners and due dates (e.g. firmware update, monitoring threshold, runbook change)

---
*Written from NetApp documentation and incident-coordination practice. Commands to be validated on an FSx for ONTAP lab.*
