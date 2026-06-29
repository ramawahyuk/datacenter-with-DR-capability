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

<img width="552" height="400" alt="image" src="https://github.com/user-attachments/assets/a108c32d-e67a-41b1-846e-e56c4fb093ba" />

---

### Step 3 — Select Deployment Type

Select: **Embedded Platform Services Controller**

This deploys vCenter and PSC on the same appliance — appropriate for small to medium environments.

<img width="552" height="400" alt="image" src="https://github.com/user-attachments/assets/739f7e1b-2e23-41d9-a7fa-3656df5ef7b9" />

---

### Step 4 — Configure Target ESXi Host

Enter the connection details for the ESXi host where VCSA will be installed:

| Field | Value |
|-------|-------|
| ESXi host | `192.168.1.28` |
| HTTPS Port | `443` |
| Username | `root` |
| Password | `(root password)` |

Accept the SSL certificate warning.

<img width="584" height="423" alt="image" src="https://github.com/user-attachments/assets/fedba0f3-a044-4ce1-a220-c5a8d9e34c08" />

---

### Step 5 — Configure the VCSA Virtual Machine

| Field | Value |
|-------|-------|
| VM Name | `VCSA` |
| Root Password | `(set strong password)` |


<img width="600" height="434" alt="image" src="https://github.com/user-attachments/assets/063e7e6c-dabc-4a67-bd23-f5be4518b42c" />

---


### Step 6 — Select Deployment Size

| Setting | Value |
|---------|-------|
| Deployment Size | **Tiny** |
| Storage Size | Default |

> The Tiny size supports up to 10 hosts and 100 VMs — appropriate for this lab.

<img width="615" height="445" alt="image" src="https://github.com/user-attachments/assets/fb32beff-402f-4b47-81cb-dabe40a79fcd" />


---
### Step 7 — Select Datastore

- Select the available datastore on Sapcore5.
- Enable **Thin Disk Mode** to conserve storage.

<img width="616" height="445" alt="image" src="https://github.com/user-attachments/assets/5f1f7ac2-b07a-482d-9a4b-f32e96fc5b3f" />

---

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

<img width="663" height="480" alt="image" src="https://github.com/user-attachments/assets/22ab37fd-521c-47dc-a8eb-9784a1f4c0b8" />

---

### Step 9 — Review and Install (Stage 1)

Review all configuration settings, then click **Finish** to begin Stage 1 deployment.

Stage 1 deploys the VCSA OVF template to the target host. This takes approximately 5–15 minutes.

---

## Stage 2 — Configure the Appliance

Stage 2 starts automatically after Stage 1 completes.

### Step 1 — Introduction

Click **Next** to proceed to Stage 2 configuration.

<img width="642" height="445" alt="image" src="https://github.com/user-attachments/assets/eb7cec6e-75f7-4fa8-b7e7-7314f183649e" />

---

### Step 2 — Configure NTP

| Field | Value |
|-------|-------|
| NTP Servers | `1.id.ntp.pool.org`, `2.id.ntp.pool.org` |
| SSH Access | Enabled (for lab purposes) |

<img width="693" height="480" alt="image" src="https://github.com/user-attachments/assets/c539200a-532b-4605-9839-6e7198bf30dd" />

---

### Step 3 — Configure SSO

| Field | Value |
|-------|-------|
| SSO Domain Name | `vsphere.local` |
| SSO Site Name | `default-first-site` |
| SSO Password | `(set strong password)` |

<img width="657" height="455" alt="image" src="https://github.com/user-attachments/assets/c1870782-60ab-4947-9911-d478df142361" />

---

### Step 4 — Review and Finish

Review the complete configuration and click **Finish** to begin Stage 2.

<img width="737" height="510" alt="image" src="https://github.com/user-attachments/assets/1fd0bc05-9085-4c32-802e-832811443fe5" />


---

Stage 2 configures all vCenter services. This takes approximately 10–20 minutes.

When complete, the installer displays the VCSA management URL.


<img width="670" height="400" alt="image" src="https://github.com/user-attachments/assets/dc4f653a-cd56-475d-a8f4-59056a8265c4" />


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
