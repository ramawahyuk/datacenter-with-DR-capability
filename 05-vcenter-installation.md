# vCenter Server Appliance (VCSA) 6.7 Installation Guide

This guide covers the two-stage deployment of VMware vCenter Server Appliance (VCSA) 6.7.

---

## Overview

| Setting | Value |
|---------|-------|
| VCSA Version | 6.7 |
| Target Host | Sapcore5 / 192.168.1.28 |
| VCSA IP Address | 192.168.1.100 |
| VCSA FQDN | vsystem.helpy.com |
| Deployment Size | Tiny |
| Storage Mode | Thin Disk |
| Domain | helpy.com |
| SSO Domain | vsphere.local |
| NTP Server | 1.id.ntp.pool.org, 2.id.ntp.pool.org |
| DNS | 192.168.1.10 |

---

## Prerequisites

1. Domain Controller is running and DNS is functional (`192.168.1.10`)
2. ESXi host `Sapcore5` (192.168.1.28) is installed and accessible
3. Windows Server 2012 R2 (vCenter client host) is joined to the `helpy.com` domain
4. VCSA 6.7 ISO is downloaded and extracted

---

## Stage 1 — Deploy the Appliance

### Step 1 — Launch the VCSA Installer

On the Windows Server 2012 R2 client:

1. Extract the VCSA 6.7 ISO.
2. Navigate to: `vcsa-ui-installer → win32`
3. Run `installer.exe`
4. Select **Install** from the welcome screen.

### Step 2 — Accept License Agreement

Read and accept the VMware End User License Agreement.

### Step 3 — Select Deployment Type

Select: **Embedded Platform Services Controller**

This deploys vCenter and PSC on the same appliance — appropriate for small to medium environments.

### Step 4 — Configure Target ESXi Host

Enter the connection details for the ESXi host where VCSA will be installed:

| Field | Value |
|-------|-------|
| ESXi host | `192.168.1.28` |
| HTTPS Port | `443` |
| Username | `root` |
| Password | `(root password)` |

Accept the SSL certificate warning.

### Step 5 — Configure the VCSA Virtual Machine

| Field | Value |
|-------|-------|
| VM Name | `VCSA` |
| Root Password | `(set strong password)` |

### Step 6 — Select Deployment Size

| Setting | Value |
|---------|-------|
| Deployment Size | **Tiny** |
| Storage Size | Default |

> The Tiny size supports up to 10 hosts and 100 VMs — appropriate for this lab.

### Step 7 — Select Datastore

- Select the available datastore on Sapcore5.
- Enable **Thin Disk Mode** to conserve storage.

### Step 8 — Configure Network Settings

| Field | Value |
|-------|-------|
| Network | VM Network |
| IP Version | IPv4 |
| IP Assignment | Static |
| IP Address | `192.168.1.100` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.1.1` |
| DNS Servers | `192.168.1.10` |
| FQDN | `vsystem.helpy.com` |

### Step 9 — Review and Install (Stage 1)

Review all configuration settings, then click **Finish** to begin Stage 1 deployment.

Stage 1 deploys the VCSA OVF template to the target host. This takes approximately 5–15 minutes.

---

## Stage 2 — Configure the Appliance

Stage 2 starts automatically after Stage 1 completes.

### Step 1 — Introduction

Click **Next** to proceed to Stage 2 configuration.

### Step 2 — Configure NTP

| Field | Value |
|-------|-------|
| NTP Servers | `1.id.ntp.pool.org`, `2.id.ntp.pool.org` |
| SSH Access | Enabled (for lab purposes) |

### Step 3 — Configure SSO

| Field | Value |
|-------|-------|
| SSO Domain Name | `vsphere.local` |
| SSO Site Name | `default-first-site` |
| SSO Password | `(set strong password)` |

### Step 4 — Review and Finish

Review the complete configuration and click **Finish** to begin Stage 2.

Stage 2 configures all vCenter services. This takes approximately 10–20 minutes.

When complete, the installer displays the VCSA management URL.

---

## Accessing vCenter Server

After deployment:

- **vSphere Web Client:** `https://vsystem.helpy.com/ui`
- **VMware Appliance Management:** `https://vsystem.helpy.com:5480`

Log in with:
- **Username:** `administrator@vsphere.local`
- **Password:** *(SSO password set in Stage 2)*

---

## Post-Deployment — Join to Domain

To allow domain users to access vCenter:

1. In vSphere Web Client, go to **Administration → Single Sign-On → Configuration**.
2. Select **Identity Sources → Add Identity Source**.
3. Choose **Active Directory (Integrated Windows Authentication)**.
4. Enter domain: `helpy.com`
5. Provide domain administrator credentials.
6. Click **OK**.

---

## Post-Deployment — Create Datacenter and Clusters

```
Datacenter:  ASIA
  ├── Cluster SYS A (Sapcore1, Sapcore2, Sapcore3)
  ├── Cluster SYS B (Sapcore4)
  └── Cluster VCSA (Sapcore5)
```

1. In vSphere Web Client, right-click the root → **New Datacenter** → Name: `ASIA`
2. Right-click the datacenter → **New Cluster** → Name: `SYS A`
   - Enable **vSphere HA** and **vSphere DRS** (optional for lab)
3. Repeat for clusters `SYS B` and `VCSA`
4. Add hosts to their respective clusters via **Add Host**

---

## Troubleshooting

| Issue | Solution |
|-------|---------|
| Stage 1 fails with SSL error | Accept the certificate warning, verify ESXi host is reachable |
| Stage 2 fails at SSO | Ensure NTP sync is correct, retry with correct password |
| Cannot access vSphere Web Client | Verify DNS resolves `vsystem.helpy.com` to `192.168.1.100` |
| Domain join fails | Verify domain controller is running and DNS is correct |
