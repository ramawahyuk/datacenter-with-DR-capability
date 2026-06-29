# DR Runbook — vSphere Replication Failover

This runbook covers the procedure for activating a replicated VM on the DR site (Cluster SYS B) after a production failure.

---

## Runbook Metadata

| Field | Value |
|-------|-------|
| Runbook ID | DR-002 |
| Feature | vSphere Replication Failover |
| Technology | vSphere Replication 8.0, Site Recovery Manager |
| RTO | Manual activation — typically 5–15 minutes |
| RPO | Up to 10 minutes (configured interval) |
| Replication Target | Cluster SYS B / Sapcore4 / 192.168.1.37 |

---

## Pre-Failover Checklist

- [ ] Production site (Cluster SYS A) is confirmed unavailable or affected VM is unrecoverable
- [ ] vSphere Replication Appliance is running (`192.168.1.180`)
- [ ] Cluster SYS B (Sapcore4) is healthy and accessible
- [ ] DNS server (192.168.1.10) is running
- [ ] Last successful replication sync time is noted (check RPO status)

---

## Failover Procedure

### Step 1 — Access Site Recovery Manager

Open a browser and navigate to:
```
https://192.168.1.180/dr/
```

Or from vSphere Web Client: **Home → Site Recovery**

Log in with vCenter credentials.

### Step 2 — Verify Replication Status

1. In the Site Recovery panel, click **Replications**.
2. Check the target VM (e.g., WIN-10):
   - **Status:** Should show `OK` or last sync timestamp
   - **RPO:** Confirm it is within the 10-minute target
3. Note the last recovery point timestamp — this determines maximum data loss.

### Step 3 — Initiate Failover

1. Select the VM to recover.
2. Click **Recovery → Failover** (or **Planned Migration** for zero-downtime scenarios).
3. Select recovery point:
   - **Latest available point** (minimizes data loss)
   - Or select a specific point-in-time snapshot
4. Review the target configuration:
   - Target datastore on Sapcore4
   - Target network on Cluster SYS B
5. Click **Next** → **Finish** to begin failover.

### Step 4 — Monitor Recovery

Monitor the recovery progress in the Site Recovery dashboard. The recovered VM will boot on Cluster SYS B.

### Step 5 — Verify Recovered VM

1. In vSphere Web Client, navigate to **Cluster SYS B → Virtual Machines**.
2. Verify the recovered VM is present and powered on.
3. Open the web console to verify the OS is accessible.
4. Test application connectivity if applicable.

---

## Re-Protection (Reverse Replication)

After failover, re-protect the VM to replicate it back to Cluster SYS A when the site is recovered:

1. In Site Recovery, select the recovered VM.
2. Click **Re-Protect**.
3. Select Cluster SYS A as the new replication target.
4. Configure RPO and click **OK**.

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Cannot access `https://192.168.1.180/dr/` | DNS down or replication VM offline | Start DNS (192.168.1.10), start vSphere Replication VM |
| Replication shows `Error` status | Network connectivity lost | Check Sapcore4 and network path from SYS A to SYS B |
| RPO exceeded | Replication lag | Check replication VM performance, reduce RPO or increase bandwidth |
| Recovered VM fails to boot | Corrupted replication data | Restore from a backup instead (see DR-001) |
