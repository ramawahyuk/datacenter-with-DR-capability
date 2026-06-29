# DR Runbook — vCenter HA Failover

This runbook covers both automatic and manual vCenter High Availability failover procedures.

---

## Runbook Metadata

| Field | Value |
|-------|-------|
| Runbook ID | DR-003 |
| Feature | vCenter High Availability Failover |
| Technology | vCenter HA (Active/Passive/Witness) |
| RTO | ~29 minutes 34 seconds (observed in lab) |
| RPO | ~0 (synchronous replication between nodes) |

---

## HA Node Reference

| Node | Role | Management IP | HA Network IP |
|------|------|--------------|--------------|
| VCSA | Active | 192.168.1.100 | 192.168.0.1 |
| VCSA-Peer | Passive | — | 192.168.0.2 |
| VCSA-Witness | Witness | — | 192.168.0.3 |

---

## Automatic Failover (No Action Required)

vCenter HA performs automatic failover when the Active node becomes unavailable. No manual steps are required.

**Monitor with:**
```cmd
ping 192.168.1.100 -t
```

Watch for:
1. `Request timed out` — Active node down, failover initiated (~27 seconds)
2. `Reply from 192.168.1.100` — Passive node promoted, services starting (~1m 46s)
3. Browser shows "Failover in progress" — vSphere Web Client initializing (~25 min)
4. Browser shows login page — Full vCenter access restored (~29m 34s)

---

## Manual Failover (Planned Maintenance)

Use this procedure for planned maintenance on the Active node.

### Prerequisites

- [ ] Verify Passive node (VCSA-Peer) is healthy
- [ ] Verify Witness node (VCSA-Witness) is healthy
- [ ] Notify stakeholders of expected ~30-minute vCenter unavailability
- [ ] Ping monitoring started: `ping 192.168.1.100 -t`

### Steps

1. Log in to vSphere Web Client at `https://vsystem.helpy.com/ui`.
2. Select the **VCSA** from the inventory.
3. Go to **Configure → vCenter HA**.
4. Click **Initiate Failover**.
5. Select **Force the failover to start immediately**.
6. Click **Yes** to confirm.

### Monitoring Timeline

| Time | Expected Observation |
|------|---------------------|
| T+0 | Failover initiated |
| T+27s | Ping: `Request timed out` |
| T+1m 46s | Ping: `Reply from 192.168.1.100` |
| T+25m | Browser: Failover/initialization in progress |
| T+29m 34s | Browser: vCenter login page accessible |

### Post-Failover Verification

- [ ] Log in to vSphere Web Client successfully
- [ ] All hosts visible in inventory
- [ ] All VMs show correct power state
- [ ] Check **Configure → vCenter HA** — verify new Active/Passive node assignments
- [ ] Node that was Active is now Passive (IPs 192.168.0.1 ↔ 192.168.0.2 swap roles)

---

## Restore HA After Failover

After failover, the former Active node becomes the new Passive. To rebalance:

1. Go to **Configure → vCenter HA**.
2. Click **Initiate Failover** again to return the original node to Active status (if needed).
3. Or leave the current Passive/Active assignment — either node can serve as Active.

---

## HA Health Check Procedure

Perform regularly to ensure HA is healthy before a real disaster:

1. Log in to vSphere Web Client.
2. Select VCSA → **Configure → vCenter HA**.
3. Verify:
   - **State:** Healthy
   - All 3 nodes: Connected
4. If any node shows `Not Connected`:
   - Power on the affected VCSA VM
   - Wait 2–3 minutes for reconnection
   - If still disconnected, check network connectivity between HA nodes

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Failover never completes | Witness node offline | Power on VCSA-Witness VM |
| vCenter inaccessible after failover for >45 minutes | Passive node issue | SSH to 192.168.1.100, check `/var/log/vmware/vpxd/` |
| Both nodes show Active | Split-brain scenario | Power off the Witness briefly to force quorum re-election |
| HA shows Degraded | Passive node unreachable | Power on VCSA-Peer, wait for sync to complete |
