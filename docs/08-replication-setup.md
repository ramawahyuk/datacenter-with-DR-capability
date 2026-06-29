# vSphere Replication 8.0 Setup Guide

This guide covers the deployment and configuration of VMware vSphere Replication for VM-level asynchronous replication from Cluster SYS A to Cluster SYS B.

---

## Overview

| Setting | Value |
|---------|-------|
| Replication Software | vSphere Replication 8.0.0 |
| Source Cluster | SYS A (Sapcore1 – 192.168.1.34) |
| Target Cluster | SYS B (Sapcore4 – 192.168.1.37) |
| Replication Appliance IP | 192.168.1.180 |
| Replication Appliance Hostname | Photon-machine |
| RPO (Recovery Point Objective) | 10 minutes |
| VRM Site Name | vSphere_SR |

---

## Replication Appliance Specifications

| Setting | Value |
|---------|-------|
| vCPU | 2 |
| Storage Format | Thin Provisioning |
| Datastore | datastore1 |
| Network | PortGas |
| IP Protocol | IPv4 |
| IP Allocation | Static |
| IP Address | 192.168.1.180 |
| NTP Server | 1.id.ntp.pool.org, 2.id.ntp.pool.org |
| DNS | 192.168.1.10 |

---

## Installer Files Required

<img width="598" height="301" alt="image" src="https://github.com/user-attachments/assets/38b0f90e-18cc-4ab4-8766-ba393e38e106" />


| File | Purpose |
|------|---------|
| `vSphere_Replication_OVF10.cert` | Certificate file |
| `vSphere_Replication_OVF10.ovf` | OVF descriptor |
| `vSphere_Replication_support.vmdk` | Support disk |
| `vSphere_Replication_system.vmdk` | System disk |

---

## Part 1 — Deploy vSphere Replication OVF

### Step 1 — Launch OVF Deployment

1. In vSphere Web Client, right-click **Sapcore1** (192.168.1.34).
2. Select **Deploy OVF Template**.

<img width="476" height="324" alt="image" src="https://github.com/user-attachments/assets/cd629523-6abd-4184-8c48-25569d979608" />


3. Select **Local file** and upload all four installer files listed above.

<img width="601" height="352" alt="image" src="https://github.com/user-attachments/assets/fe6c2c24-969f-4964-a0a2-2e235d8d7d6c" />


4. Click **Next**.

### Step 2 — Name and Location

| Field | Value |
|-------|-------|
| Name | `vSphere-Replication` |
| Location | `ASIA` datacenter → `SYS A` cluster |

### Step 3 — Select Target Host

Select **Sapcore1 / 192.168.1.34** as the target host.

### Step 4 — Review OVF Details

Review the OVF template details and accept the license agreement.

### Step 5 — Configuration

| Setting | Value |
|---------|-------|
| Deployment Configuration | **2 vCPUs** |

### Step 6 — Storage

| Setting | Value |
|---------|-------|
| Disk Format | **Thin Provision** |
| Datastore | `datastore1` |

### Step 7 — Network Mapping

| Setting | Value |
|---------|-------|
| Source Network | VM Network |
| Destination | `PortGas` |

### Step 8 — Customize Template

<img width="753" height="439" alt="image" src="https://github.com/user-attachments/assets/bb716a4a-c78a-47f8-980b-6cd89bac422c" />


| Field | Value |
|-------|-------|
| IP Address | `192.168.1.180` |
| Netmask | `255.255.255.0` |
| Gateway | `192.168.1.1` |
| DNS | `192.168.1.10` |
| NTP Server | `1.id.ntp.pool.org` |
| Hostname | `Photon-machine` |
| Root Password | *(set strong password)* |

### Step 9 — Complete Deployment

Review the summary and click **Finish** to deploy the OVF.

---

## Part 2 — Configure vSphere Replication Appliance

### Step 1 — Power On the Appliance

After deployment, power on the **vSphere-Replication** VM.

### Step 2 — Access the Appliance Configuration UI

Open a browser and navigate to:
```
https://192.168.1.180:5480
```

### Step 3 — Configure the Appliance

On the **Configuration** tab, enter:

| Field | Value |
|-------|-------|
| Mode | Configure using the embedded database |
| LookupService Address | `vsystem.helpy.com` |
| SSO Administrator | `Administrator@vsphere.local` |
| SSO Password | *(vCenter SSO password)* |
| VRM Host | `192.168.1.180` |
| VRM Site Name | `vSphere_SR` |
| vCenter Server Address | `vsystem.helpy.com` |
| vCenter Server Port | `80` |
| Admin Email | `root@192.168.1.180` |

Click **Save and Restart Service**.

### Step 4 — Verify Services

After saving, verify the following services are running:

| Service | Expected Status |
|---------|----------------|
| VRM | Running |
| Tomcat | Running |

If both services show as running, vSphere Replication is successfully deployed.

---

## Part 3 — Configure VM Replication

### Step 1 — Access Site Recovery

In vSphere Web Client, go to **Home → Site Recovery**, or navigate directly to:
```
https://192.168.1.180/dr/
```

### Step 2 — Create a New Replication

1. In the **Site Recovery** panel, click **New** to create a replication.
2. Select the VM to replicate:
   - **WIN-10** (located in Cluster SYS A)
3. Click **Next**.

### Step 3 — Select Target vCenter

1. Select the target vCenter for replication.
2. Choose **Auto-assign vSphere Replication Server** (only one replication server in this lab).
3. Click **Next**.

### Step 4 — Configure Target Storage

| Setting | Value |
|---------|-------|
| Disk Format | **Thin Provision** |
| VM Storage Policy | **Datastore Default** |
| Target Datastore | Select the datastore on Sapcore4 |

### Step 5 — Set RPO

| Setting | Value |
|---------|-------|
| RPO | **10 minutes** |

> The Recovery Point Objective (RPO) of 10 minutes means the maximum data loss in the event of a disaster is 10 minutes worth of changes.

### Step 6 — Review and Finish

Review the replication configuration and click **Finish**.

The initial synchronization will begin. Duration depends on VM size.

In this lab, a Windows 10 VM replication completed in **6 minutes 30 seconds**.

---

## Verification

After replication is configured:

1. Go to **Site Recovery → Replications**.
2. Verify the VM shows **Status: OK** and **RPO: Met**.
3. Go to **Cluster SYS B → VMs** and verify the replicated VM appears.

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Cannot access `https://192.168.1.180/dr/` | DNS not working | Verify DNS server (192.168.1.10) is running |
| Replication fails immediately | Target host (Sapcore4) is down | Verify Sapcore4 is powered on and reachable |
| Services not starting in appliance | Incorrect LookupService address | Verify VCSA FQDN `vsystem.helpy.com` resolves correctly |
| SSL certificate error | Untrusted cert | Accept the certificate in the browser or add it to trusted roots |
