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
3. Under **Server Roles**, expand **File and Storage Services → File and iSCSI Services**.

<img width="474" height="334" alt="image" src="https://github.com/user-attachments/assets/178b4db1-0f36-4789-8d94-f8ef88409ad8" />

4. Check **Server for NFS**.
5. Also check **Client for NFS** under Features.
6. Click **Install**.

<img width="429" height="429" alt="image" src="https://github.com/user-attachments/assets/e4d3b30e-efa0-4892-a69c-bf1fa434b6db" />

### Step 2 — Create NFS Directory

```cmd
# Create the NFS share directory
mkdir E:\NFS_STRG01
```

### Step 3 — Configure NFS Sharing

1. Open **File Explorer** → right-click `E:\NFS_STRG01` → **Properties**.

<img width="417" height="613" alt="image" src="https://github.com/user-attachments/assets/9ae6c225-98ad-48c5-9ec5-8617d0f0a0c6" />


2. Go to the **NFS Sharing** tab.
3. Click **Manage NFS Sharing**.
4. Click Permissions to add host or client.

<img width="421" height="610" alt="image" src="https://github.com/user-attachments/assets/0e2e93d7-8585-4db7-89e7-eae410295d11" />


5. add the ESXi hosts:

<img width="567" height="615" alt="image" src="https://github.com/user-attachments/assets/5dbace40-3c87-4be9-9ea2-6ed880e76963" />




| Setting | Value |
|---------|-------|
| Type of Access | Read-Write |
| Allow root access | Yes |
| Hosts allowed | `192.168.1.34`,`192.168.1.35`, `192.168.1.36`, `192.168.1.100`|

7. Click **OK** → **Apply** → **OK**.

---

## Part 2 — Add NFS Datastore to vSphere

### Step 1 — Add NFS Datastore in vCenter

1. In vSphere Web Client, go to **Storage → Datastores**.
2. Click **New Datastore**.

<img width="762" height="440" alt="image" src="https://github.com/user-attachments/assets/ff331086-5e3b-417a-8dc4-9f367fdf9fdf" />


3. Select location: **ASIA** datacenter.
4. Select datastore type: **NFS**.

<img width="759" height="441" alt="image" src="https://github.com/user-attachments/assets/84c1ffe9-d3ea-4da4-8b90-2d1b0f16466b" />


5. Select NFS version: **NFS 3**.

<img width="734" height="425" alt="image" src="https://github.com/user-attachments/assets/a617e516-fda3-4b58-83d3-a8fb3064e6c5" />


6. Configure datastore 

<img width="568" height="331" alt="image" src="https://github.com/user-attachments/assets/0989f3ab-06b0-42c0-b220-fe024acca58a" />


| Field | Value |
|-------|-------|
| Datastore Name | `NFS_STRG2` |
| Folder | `E:\NFS_STRG01` |
| Server | `192.168.1.99` |

7. Select the hosts that should have access:
   - `Sapcore2 / 192.168.1.34`
   - `Sapcore2 / 192.168.1.35`
   - `Sapcore3 / 192.168.1.36`

<img width="596" height="347" alt="image" src="https://github.com/user-attachments/assets/35219b76-8fad-4a6d-a788-8329f572c919" />

9. Review and click **Finish**.

<img width="599" height="349" alt="image" src="https://github.com/user-attachments/assets/6e7a160e-08df-4090-8606-51e32ee1b0dd" />


### Step 2 — Verify NFS Datastore

After creation:

<img width="772" height="382" alt="image" src="https://github.com/user-attachments/assets/df340f7e-0895-4070-9989-6b1586a562fa" />


1. Go to **Storage → Datastores**.
2. Verify `NFS-FT-Storage` appears with ~60 GB capacity.
3. All host has been registered `Sapcore1`, `Sapcore2` and `Sapcore3` should show as connected hosts.

<img width="833" height="159" alt="image" src="https://github.com/user-attachments/assets/fc5da02e-e491-4271-a0f6-b831ed67864d" />

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
3. Select the **datastore** for the Secondary VM: `NFS_STRG/NFS_STRG2`

<img width="774" height="419" alt="image" src="https://github.com/user-attachments/assets/6abf0771-121d-48a1-8989-1f459288abdb" />

4. Select the **host** for the Secondary VM: `Sapcore3 / 192.168.1.36`

<img width="775" height="371" alt="image" src="https://github.com/user-attachments/assets/1e5d5fb8-73ac-490a-8726-0e904b8f641b" />


5. Click **Next** → **Finish**.

<img width="776" height="367" alt="image" src="https://github.com/user-attachments/assets/04769030-558f-45f9-bc13-320c08621028" />


### Step 4 — Verify FT Status

After enabling:

<img width="609" height="549" alt="image" src="https://github.com/user-attachments/assets/3af0222e-938f-420e-a69c-68d6c728a68c" />


1. Select **WIN-7 FT** in the VM inventory.
2. In the **Summary** tab, verify:
   - **Fault Tolerance:** Protected

<img width="674" height="228" alt="image" src="https://github.com/user-attachments/assets/43ca379f-8696-49d6-9498-2fe2cc3f0572" />


   - **Primary VM:** on Sapcore2

<img width="669" height="251" alt="image" src="https://github.com/user-attachments/assets/85d1388e-8975-4240-b35a-31a849567d90" />

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
