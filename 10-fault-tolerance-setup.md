# VMware Fault Tolerance Setup Guide

This guide covers the configuration of VMware Fault Tolerance (FT) to provide zero-downtime protection for critical virtual machines.

---

## Overview

VMware Fault Tolerance creates a live shadow copy (Secondary VM) of a protected VM (Primary VM), running in lock-step on a different host. If the primary host fails, the Secondary VM instantly takes over with **zero downtime and zero data loss**.

| Setting | Value |
|---------|-------|
| Protected VM | WIN-7 FT |
| Primary Host | Sapcore2 / 192.168.1.35 |
| Secondary Host | Sapcore3 / 192.168.1.36 |
| Shared Storage Type | NFS (Windows Server NFS role) |
| NFS Storage Size | 60 GB |
| FT Status (success) | Protected |

---

## Why NFS is Required

VMware Fault Tolerance requires **shared storage** accessible by both the primary and secondary hosts. In this lab, a Windows Server is configured as an NFS server to provide shared storage.

> Local datastores and vSAN are not supported for Fault Tolerance in this version.

---

## Part 1 — Configure Windows NFS Server

On **BACKUPSRV** (192.168.1.99) or a dedicated Windows Server:

### Step 1 — Install Server for NFS Role

1. Open **Server Manager → Add Roles and Features**.
2. Under **Server Roles**, expand **File and Storage Services → File and iSCSI Services**.
3. Check **Server for NFS**.
4. Also check **Client for NFS** under Features.
5. Click **Install**.

### Step 2 — Create NFS Directory

```cmd
# Create the NFS share directory
mkdir C:\nfs_share
```

### Step 3 — Configure NFS Sharing

1. Open **File Explorer** → right-click `C:\nfs_share` → **Properties**.
2. Go to the **NFS Sharing** tab.
3. Click **Manage NFS Sharing**.
4. Check **Share this folder**.
5. Share name: `nfs_share`
6. Click **Permissions** to add the ESXi hosts:

| Setting | Value |
|---------|-------|
| Type of Access | Read-Write |
| Allow root access | Yes |
| Hosts allowed | `192.168.1.35`, `192.168.1.36` (Sapcore2 and Sapcore3) |

7. Click **OK** → **Apply** → **OK**.

---

## Part 2 — Add NFS Datastore to vSphere

### Step 1 — Add NFS Datastore in vCenter

1. In vSphere Web Client, go to **Storage → Datastores**.
2. Click **New Datastore**.
3. Select location: **ASIA** datacenter.
4. Select datastore type: **NFS**.
5. Select NFS version: **NFS 3**.
6. Configure:

| Field | Value |
|-------|-------|
| Datastore Name | `NFS-FT-Storage` |
| Folder | `/nfs_share` |
| Server | `192.168.1.99` |

7. Select the hosts that should have access:
   - `Sapcore2 / 192.168.1.35`
   - `Sapcore3 / 192.168.1.36`

8. Review and click **Finish**.

### Step 2 — Verify NFS Datastore

After creation:

1. Go to **Storage → Datastores**.
2. Verify `NFS-FT-Storage` appears with ~60 GB capacity.
3. Both Sapcore2 and Sapcore3 should show as connected hosts.

---

## Part 3 — Enable Fault Tolerance on VM

### Step 1 — Migrate VM to NFS Storage

Before enabling FT, the VM's files must reside on the shared NFS datastore:

1. Right-click **WIN-7 FT** → **Migrate**.
2. Select **Change storage only**.
3. Select `NFS-FT-Storage` as the destination.
4. Click **Finish**.

### Step 2 — Verify vSphere HA is Enabled

Fault Tolerance requires vSphere HA to be enabled on the cluster:

1. Select **Cluster SYS A**.
2. Go to **Configure → vSphere Availability**.
3. Verify **vSphere HA** is turned on.

### Step 3 — Enable Fault Tolerance

1. Right-click **WIN-7 FT** in the VM inventory.
2. Select **Fault Tolerance → Turn On Fault Tolerance**.
3. Select the **datastore** for the Secondary VM: `NFS-FT-Storage`
4. Select the **host** for the Secondary VM: `Sapcore3 / 192.168.1.36`
5. Click **Next** → **Finish**.

### Step 4 — Verify FT Status

After enabling:

1. Select **WIN-7 FT** in the VM inventory.
2. In the **Summary** tab, verify:
   - **Fault Tolerance:** Protected
   - **Primary VM:** on Sapcore2
   - **Secondary VM:** on Sapcore3

---

## How Fault Tolerance Works

```
Normal operation:
┌─────────────┐         ┌─────────────┐
│  Sapcore2   │◄───────►│  Sapcore3   │
│             │ FT Log  │             │
│ WIN-7 FT    │ stream  │ WIN-7 FT    │
│ (Primary)   │         │ (Secondary) │
│ RUNNING     │         │ IN SYNC     │
└─────────────┘         └─────────────┘

After Sapcore2 failure:
┌─────────────┐         ┌─────────────┐
│  Sapcore2   │  (DOWN) │  Sapcore3   │
│             │         │             │
│             │         │ WIN-7 FT    │
│             │         │ (Promoted)  │
│             │         │ RUNNING     │
└─────────────┘         └─────────────┘
```

When the primary host fails:
- The Secondary VM is **instantly promoted** to Primary — no restart, no data loss.
- A new Secondary VM is then provisioned on another available host.

---

## Testing Fault Tolerance Failover

### Test Procedure

1. Verify **WIN-7 FT** is running on Sapcore2 (Primary) and Sapcore3 (Secondary).
2. Open the **web console** of WIN-7 FT to monitor it during the test.
3. Shut down **Sapcore2**:
   - Right-click Sapcore2 → **Power → Shut Down**
4. Observe:
   - The red alarm icon appears on Sapcore2.
   - WIN-7 FT in the web console **remains active without interruption**.
   - Sapcore3 now hosts the Primary VM.
   - Sapcore2 (when it comes back) shows the Secondary VM role.

### Expected Results

| Check | Expected Result |
|-------|----------------|
| WIN-7 FT during host failure | Continues running — zero downtime |
| FT status after failover | Secondary promoted to Primary |
| Previous Primary host comes back | New Secondary VM created |

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| `Turn On Fault Tolerance` is grayed out | vSphere HA not enabled | Enable HA on the cluster first |
| FT status shows `Not Protected` | VM not on shared storage | Migrate VM to NFS datastore |
| Cannot select secondary host | Host not compatible | Ensure both hosts are the same CPU family and vSphere version |
| FT fails with vSAN | vSAN not supported for FT | Use NFS or VMFS on shared iSCSI instead |
| VM performance degrades with FT | FT logging overhead | Ensure dedicated network for FT logging traffic (VMkernel) |
