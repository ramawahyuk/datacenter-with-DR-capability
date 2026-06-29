# Disaster Scenario Tests

This document covers end-to-end disaster simulation tests performed to validate the DR objectives of the lab.

---

## Scenario 1 — Hardware Failure (Host CPU Overload)

### Business Problem
A production server experiences a sudden CPU spike to 99%, causing the host to hang and requiring an emergency shutdown. Virtual machines on the affected host must remain available.

### Pre-Test State

| Component | Location | Status |
|-----------|----------|--------|
| WIN-7 FT (Primary) | Sapcore2 / 192.168.1.35 | Running |
| WIN-7 FT (Secondary) | Sapcore3 / 192.168.1.36 | In sync |
| Fault Tolerance Status | WIN-7 FT | Protected |

### Test Steps

1. Verify WIN-7 FT is running on Sapcore2 (Primary host).
2. Open the web console of WIN-7 FT to monitor continuity.
3. Simulate CPU overload — shut down Sapcore2:
   - Right-click Sapcore2 → **Power → Shut Down**
4. Observe VM behavior in the web console.
5. Check the inventory — verify WIN-7 FT has moved to Sapcore3.

### Results

| Check | Result |
|-------|--------|
| WIN-7 FT web console during shutdown | ✅ Continues running — no interruption |
| WIN-7 FT location after failure | ✅ Moved to Sapcore3 automatically |
| Sapcore2 inventory (after shutdown) | ✅ Empty — Primary VM migrated away |
| Downtime experienced | **0 seconds** |

### Conclusion

VMware Fault Tolerance successfully mitigated a hardware failure scenario. The virtual machine remained running throughout the entire host failure event without any downtime or data loss.

---

## Scenario 2 — Software Failure (OS Corruption / VM Failure)

### Business Problem
A virtual machine running Windows experiences a critical software failure causing the OS to become unusable. The VM must be recovered to a working state from a replicated copy.

### Pre-Test State

| Component | Status |
|-----------|--------|
| WIN-10 VM on Cluster SYS A | Running |
| WIN-10 VM on Cluster SYS B | Not present |
| vSphere Replication | Configured with 10-minute RPO |
| Cluster SYS B (Sapcore4) | Running |

### Test Steps

1. Verify WIN-10 does not exist on Cluster SYS B.
2. Initiate replication via Site Recovery Manager (`https://192.168.1.180/dr/`).
3. Monitor synchronization progress.
4. After replication completes, verify WIN-10 exists on Cluster SYS B.
5. Compare configuration of both VMs to confirm they are identical.

### Results

| Check | Result |
|-------|--------|
| WIN-10 on Cluster SYS B before replication | ❌ Not present |
| Replication duration | **6 minutes 30 seconds** |
| WIN-10 on Cluster SYS B after replication | ✅ Present |
| VM configurations identical (SYS A vs SYS B) | ✅ Confirmed |
| RPO achieved | ✅ Within 10-minute target |

### Conclusion

vSphere Replication successfully replicated the production VM to the DR site (Cluster SYS B) within the defined RPO target. In the event of a software failure, the replicated VM on Cluster SYS B can be powered on as a recovery copy.

---

## DR Objectives Summary

| Objective | Technology | Achieved | Notes |
|-----------|-----------|---------|-------|
| Zero-downtime hardware failover | Fault Tolerance | ✅ Yes | 0 seconds downtime |
| VM recovery to DR site | vSphere Replication | ✅ Yes | 6m 30s sync time, 10-min RPO |
| vCenter platform recovery | vCenter HA | ✅ Yes | ~29m 34s RTO |
| Data backup and restore | VMware Appliance Management + FTP | ✅ Yes | Full vCenter restore validated |
| Cost minimization | Virtualization (nested VMs) | ✅ Yes | No physical DR hardware required |

---

## RTO / RPO Summary

| DR Feature | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) |
|------------|-------------------------------|-------------------------------|
| Fault Tolerance | **0 seconds** | **0 seconds** |
| vCenter HA Failover | **~29 minutes 34 seconds** | ~0 seconds (sync replication) |
| vSphere Replication | Manual activation required | **10 minutes** (configurable) |
| Backup / Restore | ~20–30 minutes (manual process) | Last backup timestamp |
