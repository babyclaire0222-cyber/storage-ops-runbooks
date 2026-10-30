# storage-ops-runbooks

Operational runbooks and checklists for running NetApp ONTAP storage and Brocade Fibre Channel SAN fabrics, written for L1/L2 storage operations.

Each runbook follows the same structure: purpose and scope, pre-checks, steps with commands, verification, rollback and communications.

## Contents

| File | Covers |
|---|---|
| [runbooks/p1-incident-escalation.md](runbooks/p1-incident-escalation.md) | P1 response: first 15 minutes, evidence capture, NetApp case and SAM escalation, hardware replacement, stakeholder updates, post-incident review |
| [runbooks/ontap-ndu-upgrade.md](runbooks/ontap-ndu-upgrade.md) | Non-disruptive ONTAP upgrade on a 2-node HA pair with the `cluster image` workflow |
| [runbooks/san-zoning-change.md](runbooks/san-zoning-change.md) | Single-initiator zoning of a new host on dual Brocade fabrics and LUN presentation in ONTAP |
| [checklists/preventive-maintenance.md](checklists/preventive-maintenance.md) | Daily, weekly, monthly and quarterly checks for ONTAP health, capacity, replication and SAN fabrics |

## Status

Written from NetApp ONTAP and Brocade FOS documentation and my experience coordinating vendors and site teams as a project engineer. ONTAP commands are being validated on an Amazon FSx for NetApp ONTAP lab ([ontap-hybrid-storage-lab](https://github.com/babyclaire0222-cyber/ontap-hybrid-storage-lab), in progress). Brocade commands are unvalidated because no switch simulator is available.

## Related projects

- **ontap-hybrid-storage-lab** (in progress): Terraform-built FSx for ONTAP with NFS, iSCSI, Snapshots and SnapMirror DR testing.
- **storage-health-check** (planned): automated daily checks from `preventive-maintenance.md`.

## Author

Claire · [GitHub](https://github.com/babyclaire0222-cyber)
