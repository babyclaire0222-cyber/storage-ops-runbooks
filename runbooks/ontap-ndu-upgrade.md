# ONTAP Non-Disruptive Upgrade (NDU) on a 2-Node HA Pair

## Purpose and scope

Upgrade ONTAP on a two-node FAS/AFF HA pair without taking data offline, using the automated `cluster image` workflow. During the upgrade each node is taken over by its partner, updated and given back in turn, so hosts keep access through the surviving node.

Out of scope: MetroCluster, clusters larger than two nodes, and SAN switch firmware (see `san-zoning-change.md` for fabric changes).

## Planning (1–2 weeks before)

1. **Choose the target version.** Generate an upgrade plan in NetApp Digital Advisor (Upgrade Advisor) for this cluster.
2. **Check compatibility.** Confirm hosts, HBAs, multipath software, VMware ESXi versions and SAN switches are supported on the target version in the NetApp Interoperability Matrix Tool (IMT).
3. **Check the upgrade path.** Some versions need an intermediate hop; follow the path Digital Advisor gives.
4. **SnapMirror.** If this cluster has SnapMirror relationships, upgrade the **destination** cluster before the source.
5. **Change approval.** Raise the change request with the window, impact statement ("no outage expected; brief path failover on hosts") and rollback plan. Notify application owners.
6. **Stage the image** on an HTTP/HTTPS server the cluster can reach.

## Pre-checks (start of the change window)

Record the output of each command in the change ticket.

```
cluster show
storage failover show
system health status show
system health alert show
storage aggregate show -state !online
volume show -state !online
network interface show -is-home false
snapmirror show -fields state,status,lag-time
cluster image show
```

**Stop and do not proceed if:**
- `storage failover show` does not report takeover possible on both nodes,
- there are active health alerts or failed disks,
- any aggregate or volume is offline,
- hosts report fewer than two paths to their LUNs.

## Steps

1. **Suppress automatic support cases** during the window:
   ```
   system node autosupport invoke -node * -type all -message MAINT=4h
   ```
2. **Download the image to the cluster:**
   ```
   cluster image package get -url http://<server>/<ontap_image>.tgz
   cluster image package show-repository
   ```
3. **Validate:**
   ```
   cluster image validate -version <target_version>
   ```
   Fix every error and review every warning before continuing.
4. **Estimate duration:**
   ```
   cluster image update -version <target_version> -estimate-only
   ```
5. **Start the update:**
   ```
   cluster image update -version <target_version>
   ```
6. **Monitor until complete:**
   ```
   cluster image show-update-progress
   ```
   - If a step fails, the update pauses. Check the error, fix it, then `cluster image resume-update`.
   - To stop deliberately between nodes: `cluster image pause-update`.

## Verification

```
cluster image show
storage failover show
system health status show
system health alert show
network interface show -is-home false
snapmirror show -fields state,status,lag-time
```

- Both nodes show the target version.
- Takeover is possible on both nodes again.
- No data LIFs are away from their home port. If any are: `network interface revert -vserver * -lif *`.
- SnapMirror relationships are healthy and transfers resume.
- Application owners confirm their services are normal.

Then end the maintenance window for AutoSupport:
```
system node autosupport invoke -node * -type all -message MAINT=end
```

## Rollback

- A paused or failed update: do not force it. Capture `cluster image show-update-progress` output, open a NetApp support case and wait for guidance.
- Reverting ONTAP to an earlier release is **not** a simple undo; it is done only with NetApp support. Plan the change so a revert is never the first option: validate thoroughly and stop at any failed pre-check.

## Communications

- Before: change notice to application owners with the window and expected impact.
- During: update the change ticket at each node's takeover and giveback.
- After: completion notice with the verification results.

---
*Written from NetApp ONTAP documentation. Commands to be validated on an FSx for ONTAP lab.*
