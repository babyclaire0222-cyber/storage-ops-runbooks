# Preventive Maintenance Checklist (NetApp ONTAP and Brocade SAN)

## Purpose and scope

Routine checks for a NetApp FAS/AFF cluster and its Brocade SAN fabrics, run per the device inventory. Record each run (date, engineer, result, follow-up ticket) in the daily check log. Anything abnormal becomes a ticket; anything matching a P1 definition follows `../runbooks/p1-incident-escalation.md`.

## Daily

### ONTAP cluster health

| Check | Command | Healthy result |
|---|---|---|
| Overall health | `system health status show` | `ok` |
| Active alerts | `system health alert show` | No alerts |
| HA status | `storage failover show` | Takeover possible on all nodes |
| Recent errors | `event log show -severity EMERGENCY,ALERT,ERROR` | Nothing new since last check |
| Failed disks | `storage disk show -broken` | None |
| Spare disks | `storage aggregate show-spare-disks` | At least the agreed number of spares per node |

### Capacity

| Check | Command | Threshold |
|---|---|---|
| Aggregate usage | `storage aggregate show -fields percent-used,state` | Warn at 80%, act at 85% |
| Volume usage | `volume show -fields percent-used` | Warn at 85%, act at 90% |
| Offline objects | `volume show -state !online` | None |

### Replication

| Check | Command | Healthy result |
|---|---|---|
| SnapMirror status | `snapmirror show -fields state,status,healthy,lag-time` | All `healthy`, lag within the agreed RPO |

### Network

| Check | Command | Healthy result |
|---|---|---|
| LIFs on home port | `network interface show -is-home false` | None |
| Ports down | `network port show -link down` | Only ports known to be unused |

### Brocade fabrics (each switch)

| Check | Command | Healthy result |
|---|---|---|
| Switch status | `switchstatusshow` | All components healthy |
| Port errors | `porterrshow` | No rising CRC or encoding errors since last check |
| Error log | `errdump` | No new critical entries |

## Weekly

- Review the week's capacity trend in Unified Manager / Data Infrastructure Insights; flag any aggregate projected to reach 85% within 90 days.
- Confirm AutoSupport is sending: `system node autosupport history show -node *`.
- Confirm Snapshot schedules ran and old Snapshots are expiring as per policy.
- Check open NetApp support cases and pending hardware replacements.

## Monthly

- Compare ONTAP, disk, shelf and Service Processor firmware versions against NetApp Digital Advisor recommendations; raise changes for any that are out of date or affected by a published advisory.
- Review Digital Advisor risk and wellness findings.
- Review Brocade FOS versions against the Interoperability Matrix for the current ONTAP release.
- Back up each switch's configuration (`configupload`).

## Quarterly

- Planned takeover and giveback test on each HA pair, in an approved maintenance window.
- SnapMirror DR test: break, verify data at the destination, resync (see the DR test log in the lab project).
- Physical inspection on site: fault LEDs, cabling and labels, rack airflow, room temperature.
- Review this checklist and update thresholds with the team.

---
*Written from NetApp ONTAP and Brocade FOS documentation. Commands to be validated on an FSx for ONTAP lab; Brocade commands unvalidated (no simulator available).*
