# vSAN Storage Configuration Guide

This guide covers the configuration of VMware vSAN (Virtual Storage Area Network) on the SYS A cluster.

---

## Overview

vSAN creates a distributed shared datastore across all hosts in a cluster by pooling their local storage. This lab uses **Hybrid vSAN** (Flash cache + HDD capacity).

| Setting | Value |
|---------|-------|
| Cluster | SYS A |
| Hosts | Sapcore1, Sapcore2, Sapcore3 |
| Switch | Distributed Switch (DstrbtdSwitch) |
| Port Group | Portgas |
| Minimum Hosts | 3 (VMware minimum requirement) |
| Cache Disk per Host | 1 × 10 GB (Flash) |
| Capacity Disk per Host | 1 × 25 GB (HDD) |
| Total Estimated Capacity | ~25 GB (usable, with default redundancy) |

---

## Step 1 — Create a Distributed Switch

A Distributed Switch (dvSwitch) is required so all three hosts share the same virtual switch configuration.

1. In vSphere Web Client, right-click the **ASIA** datacenter.
2. Select **Distributed Switch → New Distributed Switch**.
3. Configure:
   - **Name:** `DstrbtdSwitch`
   - **Version:** Compatible with ESXi 6.5
   - **Number of Uplinks:** 1
4. Click **Finish**.

---

## Step 2 — Create a Distributed Port Group

1. On the **DstrbtdSwitch**, navigate to **Configure → Topology**.
2. Click **Create a new distributed port group**.
3. Name it: `Portgas`
4. Leave defaults (VLAN type: None, Port binding: Static).
5. Click **OK**.

---

## Step 3 — Add Hosts to the Distributed Switch

1. Right-click **DstrbtdSwitch** → **Add and Manage Hosts**.
2. Select **Add hosts**.
3. Add: `Sapcore1`, `Sapcore2`, `Sapcore3`.
4. Assign each host's physical NIC as an **uplink** to the Distributed Switch.
5. Complete the wizard.

---

## Step 4 — Configure VMkernel Adapters

Create VMkernel adapters for each required service on each host.

### VMkernel Adapter Specifications

| Hostname | Port Group | Service | VMkernel Port | IP Address |
|----------|-----------|---------|--------------|------------|
| Sapcore1 | Portgas | Management | Vmk0 | 192.168.1.34 |
| Sapcore1 | Portgas | vMotion | Vmk2 | 192.168.1.80 |
| Sapcore1 | Portgas | vSAN | Vmk1 | 192.168.1.70 |
| Sapcore2 | Portgas | Management | Vmk0 | 192.168.1.35 |
| Sapcore2 | Portgas | vMotion | Vmk2 | 192.168.1.81 |
| Sapcore2 | Portgas | vSAN | Vmk1 | 192.168.1.71 |
| Sapcore3 | Portgas | Management | Vmk0 | 192.168.1.36 |
| Sapcore3 | Portgas | vMotion | Vmk2 | 192.168.1.83 |
| Sapcore3 | Portgas | vSAN | Vmk1 | 192.168.1.72 |

### Adding a VMkernel Adapter

For each host:

1. In vSphere Web Client, select the host.
2. Go to **Configure → Networking → VMkernel Adapters**.
3. Click the **+** (Add Networking) icon.
4. Select **VMkernel Network Adapter** → **Existing distributed port group**.
5. Select `Portgas`.
6. Enable the appropriate service (Management, vMotion, or vSAN).
7. Set static IP from the table above.
8. Click **Finish**.

Repeat for each VMkernel adapter on each host.

---

## Step 5 — Enable vSAN on the Cluster

### Storage Requirements per Host

| Host | Flash (Cache) Disk | Capacity Disk |
|------|--------------------|---------------|
| Sapcore1 | 1 × 10 GB | 1 × 25 GB |
| Sapcore2 | 1 × 10 GB | 1 × 25 GB |
| Sapcore3 | 1 × 10 GB | 1 × 25 GB |

### Enabling vSAN

1. In vSphere Web Client, select **Cluster SYS A**.
2. Go to **Configure → vSAN → General**.
3. Click **Configure**.
4. Select **Hybrid** mode (Flash cache + HDD capacity).
5. On the disk claiming screen, the system automatically detects and groups:
   - The 10 GB disk as **Cache Tier**
   - The 25 GB disk as **Capacity Tier**
6. Verify all three hosts have their disks correctly claimed.
7. Click **Next** → **Finish**.

---

## Step 6 — Verify vSAN Datastore

After configuration:

1. Go to **Storage → Datastores**.
2. A new datastore named **vsanDatastore** should appear.
3. All three hosts in Cluster SYS A should have access to this shared datastore.

The vsanDatastore capacity will be approximately **25 GB usable** (limited by the weakest host's capacity disk, with RAID-1 mirroring by default).

---

## vSAN Health Check

1. Select **Cluster SYS A → Monitor → vSAN → Health**.
2. Run the health check and verify all items are green.
3. Common items to check:
   - vSAN Build Recommendation
   - Network Health
   - Physical Disk Health
   - Data Health

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| vSAN will not enable | Less than 3 hosts in cluster | Add all three ESXi 6.5 hosts first |
| Disks not claimed automatically | Disk already has a partition | Use ESXi shell to wipe disk: `partedUtil mklabel /dev/diskX msdos` |
| vSAN health shows network errors | VMkernel ports not tagged for vSAN | Verify Vmk1 on each host has vSAN traffic enabled |
| vsanDatastore not visible on host | Host not in cluster | Add the host to Cluster SYS A |
