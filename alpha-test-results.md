# Alpha Testing Results — Blackbox Functional Testing

All DR features were tested using **blackbox testing methodology**, evaluating functional correctness with both valid and invalid inputs.

---

## Test Environment

| Component | Status Before Testing |
|-----------|----------------------|
| VCSA (Active) | Running |
| VCSA-Peer (Passive) | Running |
| VCSA-Witness | Running |
| Backup Server (BACKUPSRV) | Running |
| vCenter HA Services | Active |
| Sapcore1–Sapcore5 | All Running |

---

## 1. Backup Testing

### Applications Used
- VMware Workstation
- Command Prompt (CMD)
- Google Chrome
- WinSCP

### Test Case 1.1 — Scheduled Backup (Valid Data)

| Field | Input |
|-------|-------|
| Backup Location | `ftp://192.168.1.99:21/ftproot/vcenter_backup` |
| Schedule | Daily, 04:36 PM |
| Username | `Administrator` |
| Password | `Sapcore00!` |

**Result:** ✅ SUCCESS — Backup executed on schedule. Files created at:
```
C:\inetpub\ftproot\vcenter_backup\vCenter\sn_vsystem.helpy.com\
  M_6.7.0.14000_20200128-161603_
  M_6.7.0.14000_20200128-161655_
```

### Test Case 1.2 — Scheduled Backup (Invalid Data)

| Field | Input |
|-------|-------|
| Backup Location | `ftp://192.168.1.99/` (incorrect path) |
| Username | `Admin` (wrong user) |
| Password | `Sapcore` (wrong password) |

**Result:** ❌ FAIL (expected) — Failed to connect to backup server. Backup did not execute.

### Test Case 1.3 — Manual Backup (Valid Data)

| Field | Input |
|-------|-------|
| Backup Location | `ftp://192.168.1.99:21/ftproot/vcenter_backup` |
| Username | `Administrator` |
| Password | `Sapcore00!` |
| Action | Click **Backup Now** |

**Result:** ✅ SUCCESS — Backup files created:
```
M_6.7.0.14000_20191223-161713_
M_6.7.0.14000_20200106-162305_
M_6.7.0.14000_20200106-164714_
S_6.7.0.14000_20191223-165909_
```

### Test Case 1.4 — Manual Backup (Invalid Data)

| Field | Input |
|-------|-------|
| Backup Location | `ftp://192.168.1.99/` (incorrect path) |
| Username | `Admin` |
| Password | `Sapcore!` |

**Result:** ❌ FAIL (expected) — Authentication failed. No backup created.

---

> More information about Backup can see this [documentation](https://github.com/ramawahyuk/datacenter-with-DR-capability/blob/main/07-backup-configuration.md)

## 2. Restore Testing

### Applications Used
- VMware Workstation
- Command Prompt (CMD)
- Google Chrome
- VCSA 6.7 Installer

### Pre-Test State

| Component | Condition |
|-----------|-----------|
| VCSA | Not present (deleted to simulate failure) |
| VCSA-Peer | Present |
| VCSA-Witness | Present |
| Ping 192.168.1.100 | Replies (from Peer node) |

### Test Case 2.1 — Restore (Valid Data)

| Field | Input |
|-------|-------|
| Backup Location | `ftp://192.168.1.99:21/ftproot/vcenter_backup` |
| Target Host | `192.168.1.28` (Sapcore5) |
| Username | `Administrator` |
| Password | `Sapcore00!` |
| VM Name | `VCSA-1` |
| Deployment Size | Tiny |
| Storage Mode | Thin Disk |

**Result:** ✅ SUCCESS — Deleted VCSA VM was successfully restored. Network config auto-populated from backup data.

### Test Case 2.2 — Restore (Invalid Data)

| Field | Input |
|-------|-------|
| Backup Location | `ftp://192.168.1.99:21/ftproot/vcenter_backup` |
| Target Host | `192.168.1.28` |
| Username | `Administrator` |
| Password | `Sapcore00!` (wrong — simulated) |

**Result:** ❌ FAIL (expected) — Authentication error. Restore process aborted.

---

## 3. Replication Testing

### Applications Used
- VMware Workstation
- vSphere Replication 8.0.0
- Google Chrome / VCSA 6.7

### Pre-Test State

| Component | Status |
|-----------|--------|
| WIN-10 VM on Cluster SYS A | Present |
| WIN-10 VM on Cluster SYS B | Not present |
| DNS Server | Active |




### Test Case 3.1 — Access Site Recovery (Valid)

| Field | Input |
|-------|-------|
| URL | `https://192.168.1.180/dr/` |
| DNS | Active |
| vSphere Replication VM | Running |

**Result:** ✅ SUCCESS — Site Recovery Manager interface accessible.

### Test Case 3.2 — Perform VM Replication (Valid)

| Field | Input |
|-------|-------|
| Source VM | WIN-10 (Cluster SYS A) |
| Target Host | Sapcore4 (Cluster SYS B) |
| DNS | Active |

<img width="311" height="205" alt="image" src="https://github.com/user-attachments/assets/18efdd00-e9ae-4ea9-b7f6-38ad5ff24f13" />

Initial state of the WIN-10 VM is on and it is located at cluster SYS A

<img width="829" height="195" alt="image" src="https://github.com/user-attachments/assets/9095979d-954e-48d2-a231-a99e25c63fa0" />

On the Site Recovery Manager initiate syncronization between two clusters


<img width="828" height="356" alt="image" src="https://github.com/user-attachments/assets/aa43120e-758e-4d91-9b69-90f556913633" />

Then we start the recovery 

<img width="513" height="420" alt="image" src="https://github.com/user-attachments/assets/8d7fe68a-3012-4929-a38a-a1c12a370054" />

Choose the recovery site which is at SYS B

<img width="330" height="83" alt="image" src="https://github.com/user-attachments/assets/f63aec4f-bc20-4622-bc49-e9f652142034" />


**Result:** ✅ SUCCESS

| Metric | Value |
|--------|-------|
| Replication Duration | **6 minutes 30 seconds** |
| VM in Cluster SYS B | ✅ Present after replication |
| VM configurations identical | ✅ Confirmed |

### Test Case 3.3 — Replication (Invalid — DNS Down + Host Down)

| Field | Input |
|-------|-------|
| DNS Server | Not active |
| Sapcore4 | Down |

**Result:** ❌ FAIL (expected) — Could not access Site Recovery. Replication process failed.

---

## 4. vCenter HA Failover Testing

### Applications Used
- VMware Workstation
- Command Prompt (CMD)
- Stopwatch
- Google Chrome

### Pre-Test State

| Component | Status |
|-----------|--------|
| VCSA (Active) | Running |
| VCSA-Peer (Passive) | Running |
| VCSA-Witness | Running |
| Ping 192.168.1.100 | Replying |
| vCenter HA | Active |

<img width="681" height="269" alt="image" src="https://github.com/user-attachments/assets/d27c5c3a-30c7-4688-aedd-7bb8246265b8" />


### Test Case 4.1 — Initiate Failover (Valid)

Action: **Configure → vCenter HA → Initiate Failover → Force immediately → Yes**

**Observed Timeline:**

| Phase | Observation | Duration |
|-------|------------|---------|
| Failover initiated | `ping 192.168.1.100`: Request timed out | ~27 seconds |
| vCenter services up | `ping 192.168.1.100`: Reply | ~1 minute 46 seconds |
| Web Client initializing | Browser: "Failover in progress" | ~25 minutes |
| Full access restored | Browser: vCenter Web Client login page | **~29 minutes 34 seconds** |

We Initiate Failover of VCSA
<img width="444" height="252" alt="image" src="https://github.com/user-attachments/assets/47f36fa2-b2fc-48e4-b389-9e288063e918" />

Connection from vCenter going up after 1 minute 46 seconds, but vSphere UI still cannot be accessed

<img width="702" height="394" alt="image" src="https://github.com/user-attachments/assets/d638af5d-5476-4110-a624-ee49fe5a4082" />

After 4 minutes and 2 seconds the vSphere UI services already available, but still cannot use it

<img width="707" height="397" alt="image" src="https://github.com/user-attachments/assets/9c2d6897-7869-486e-a8d3-083e2fe53bd4" />

Then the vSphere UI services is up after 29 minutes 34 seconds, and we can access to it.

<img width="748" height="420" alt="image" src="https://github.com/user-attachments/assets/764d363a-2a3f-4d55-b8f0-f0e89071ed4d" />


**Result:** ✅ SUCCESS — vCenter HA failover completed. Passive node took over Active role.

Post-failover node state:
- Old Active (192.168.0.1) → now shows Passive
- Old Passive (192.168.0.2) → now shows Active

<img width="750" height="365" alt="image" src="https://github.com/user-attachments/assets/d1f35c67-939c-4f09-baaf-39304659dde5" />


### Test Case 4.2 — Failover (Invalid — All Nodes Down)

All VCSA VMs powered off.

**Result:** ❌ FAIL (expected) — No failover possible. vCenter unavailable.

---

## 5. Fault Tolerance Failover Testing

### Applications Used
- VMware Workstation
- Command Prompt (CMD)
- vSphere Web Client

### Pre-Test State

| Component | Location | Status |
|-----------|----------|--------|
| WIN-7 FT (Primary) | Sapcore2 / 192.168.1.35 | Running |
| WIN-7 FT (Secondary) | Sapcore3 / 192.168.1.36 | In sync |
| vSphere HA | Sapcore2, Sapcore3 | Connected |
| FT Status | WIN-7 FT | Protected |

### Test Case 5.1 — Hardware Failure Simulation (Valid)

Initialy the WIN-7 FT is active on `sapcore2` 

<img width="642" height="217" alt="image" src="https://github.com/user-attachments/assets/097d0ec1-19fd-489b-af21-62e77361e566" />


Action: Power off Sapcore2 (simulating hardware failure with CPU spike to 99% and the host became unresponsive/hang), therefore we need to reboot the host.

<img width="350" height="253" alt="image" src="https://github.com/user-attachments/assets/2ed4ce96-24d4-4d9f-82b7-af4cc41c492c" />


**Result:** ✅ SUCCESS

| Check | Result |
|-------|--------|
| WIN-7 FT web console during shutdown | Continues running — zero interruption |
| WIN-7 FT after Sapcore2 shutdown | Now running on Sapcore3 |
| Sapcore2 VM inventory | Empty (Primary VM migrated away) |
| Downtime | **0 seconds** |

This is the VM state after we initiate system shutdown, the VM WIN7-FT migrate to secondary hosts with zero downtime.

<img width="661" height="278" alt="image" src="https://github.com/user-attachments/assets/482804ba-66a2-4e75-a39a-6a516acf29c6" />



### Test Case 5.2 — FT Not Protected (Invalid)

VM set to FT Not Protected (Primary and Secondary both powered off).

**Result:** ❌ FAIL (expected) — FT failover did not complete. VM did not migrate.

---

## Alpha Testing Summary

| Feature | Valid Input | Invalid Input | Overall |
|---------|------------|--------------|---------|
| Backup (Scheduled) | ✅ Pass | ✅ Fail as expected | ✅ Pass |
| Backup (Manual) | ✅ Pass | ✅ Fail as expected | ✅ Pass |
| Restore | ✅ Pass | ✅ Fail as expected | ✅ Pass |
| Replication | ✅ Pass | ✅ Fail as expected | ✅ Pass |
| vCenter HA Failover | ✅ Pass | ✅ Fail as expected | ✅ Pass |
| Fault Tolerance Failover | ✅ Pass (zero downtime) | ✅ Fail as expected | ✅ Pass |

**Conclusion:** All DR features function correctly. The system meets functional requirements for backup, restore, replication, and failover capabilities.
