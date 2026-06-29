# vCenter High Availability (HA) Setup Guide

This guide covers the configuration of vCenter High Availability, providing automatic vCenter failover using a 3-node Active/Passive/Witness architecture.

---

## Overview

vCenter HA protects the vCenter Server Appliance (VCSA) against host failures and network partitions by maintaining three nodes:

| Node | Role | IP (Management) | IP (HA Network) |
|------|------|----------------|----------------|
| VCSA | Active | 192.168.1.100 | 192.168.0.1 |
| VCSA-Peer | Passive | — | 192.168.0.2 |
| VCSA-Witness | Witness | — | 192.168.0.3 |

**Key design point:** The vCenter HA network uses a **separate subnet** (`192.168.0.x`) from the management network (`192.168.1.x`).

---

## ESXi Host for vCenter HA

| Setting | Value |
|---------|-------|
| Host | Sapcore5 |
| IP Address | 192.168.1.28 |
| ESXi Version | 6.7 |
| Port Group | Management Network + VCENTER_HA |

---

## Port Group Configuration

Two port groups must exist on Sapcore5:

| Port Group | Purpose | Subnet |
|------------|---------|--------|
| `VM Network` | Management traffic | 192.168.1.x |
| `VCENTER_HA` | vCenter HA heartbeat | 192.168.0.x |

---

## Step 1 — Access vCenter HA Configuration

1. In vSphere Web Client, select the **VCSA** (vCenter Server).
2. Go to **Configure → vCenter HA**.
3. Click **Configure**.

---

## Step 2 — Select Configuration Mode

Select **Basic** mode and click **Next**.

> Basic mode automatically provisions the Passive and Witness nodes — simpler for lab environments.

---

## Step 3 — Configure the HA Network

Set IP addresses for the Passive and Witness nodes on the HA network subnet:

| Field | Value |
|-------|-------|
| Active Node — HA IP | `192.168.0.1` |
| Passive Node — HA IP | `192.168.0.2` |
| Witness Node — HA IP | `192.168.0.3` |

> **Important:** The HA IPs must be on a **different subnet** from the management network. Using `192.168.0.x` while management uses `192.168.1.x` satisfies this requirement.

---

## Step 4 — Configure Deployment for Passive and Witness Nodes

### Passive Node Configuration

| Setting | Value |
|---------|-------|
| Management Network | VM Network |
| vCenter HA Network | VCENTER_HA |
| Storage | datastore1 |
| Virtual Disk Format | Thin Provisioning |
| HA IP | 192.168.0.2 |

### Witness Node Configuration

| Setting | Value |
|---------|-------|
| Management Network | — (not required) |
| vCenter HA Network | VCENTER_HA |
| Storage | datastore1 |
| Virtual Disk Format | Thin Provisioning |
| HA IP | 192.168.0.3 |

---

## Step 5 — Review and Finish

1. Review the complete vCenter HA configuration summary.
2. Click **Finish**.

The system will:
1. Clone VCSA to create **VCSA-Peer** (Passive node)
2. Create a lightweight **VCSA-Witness** VM
3. Configure HA replication between Active and Passive nodes

This process takes approximately 10–15 minutes.

---

## Verification — View HA Status

After configuration:

1. In vSphere Web Client, select the VCSA.
2. Go to **Configure → vCenter HA**.
3. Verify:
   - **State:** Healthy
   - **Active Node:** VCSA (192.168.1.100)
   - **Passive Node:** VCSA-Peer (192.168.0.2)
   - **Witness Node:** VCSA-Witness (192.168.0.3)

---

## How vCenter HA Failover Works

When the Active node fails:

1. The Witness node detects the Active node is unavailable.
2. The Passive node is promoted to become the new Active node.
3. vCenter services restart on the Passive node.
4. Management IP (`192.168.1.100`) is assumed by the former Passive node.

**Observed RTO in this lab:**

| Phase | Time |
|-------|------|
| vCenter goes unreachable (ping timeout) | ~27 seconds |
| Services restored, ping replies | ~1 minute 46 seconds |
| vSphere Web Client initializing | ~25 minutes |
| Full vCenter access restored | **~29 minutes 34 seconds** |

---

## Manual Failover (Testing)

To manually initiate a failover for testing:

1. Select the VCSA in vSphere Web Client.
2. Go to **Configure → vCenter HA**.
3. Click **Initiate Failover**.
4. Select **Force the failover to start immediately**.
5. Click **Yes**.

To monitor:
```cmd
# From Windows client CMD:
ping 192.168.1.100 -t
```

Watch for the sequence: `Request timed out` → `Reply from 192.168.1.100`

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| HA configuration fails | HA network port group not found | Create `VCENTER_HA` port group on vSwitch0 of Sapcore5 |
| Passive node not created | Insufficient disk space | Free up space on datastore1 |
| Witness node shows `Not Connected` | Network misconfiguration | Verify `VCENTER_HA` port group connectivity |
| Failover takes too long | Normal behavior | vCenter HA RTO is expected to be 5–30 minutes |
| After failover, vSphere Web Client unreachable | vSphere Web Client still initializing | Wait 25–30 minutes for full initialization |
