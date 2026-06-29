# Backup Configuration Guide

This guide covers the setup of the Windows Server FTP backup target and vCenter scheduled/manual backups using VMware Appliance Management.

---

## Overview

| Setting | Value |
|---------|-------|
| Backup System | VMware Appliance Management (port 5480) |
| Backup Server | BACKUPSRV / 192.168.1.99 |
| Backup Protocol | FTP |
| FTP Server Software | IIS (Internet Information Services) |
| Backup Location (format) | `ftp://192.168.1.99:21/ftproot/vcenter_backup` |
| FTP User | `ftpadm` |
| Backup Type | Scheduled (daily) + Manual |

---

## Part 1 — Configure Windows Server as FTP Backup Server

### Step 1 — Install IIS with FTP Service

On **BACKUPSRV** (Windows Server 2012 R2):

1. Open **Server Manager**.
2. Click **Manage → Add Roles and Features**.
3. Select **Role-based or feature-based installation**.
4. Select the local server.
5. Under **Server Roles**, expand **Web Server (IIS)**.

<img width="552" height="392" alt="image" src="https://github.com/user-attachments/assets/878bf0af-a2b3-4d1d-8d0b-7f1c8b8866b2" />


6. Check **Web Server → FTP Server → FTP Service**.

<img width="568" height="407" alt="image" src="https://github.com/user-attachments/assets/11996981-0259-4e76-9557-83255d7c7308" />


7. Click **Next** through Features, then **Install**.
8. Wait for installation to complete.

<img width="573" height="406" alt="image" src="https://github.com/user-attachments/assets/e3702b9c-a9a6-4ec2-a7a6-d4b3f96fd5f2" />

---


### Step 2 — Create FTP User

1. Open **Computer Management** (right-click Start → Computer Management).
2. Navigate to **Local Users and Groups → Users**.

<img width="587" height="419" alt="image" src="https://github.com/user-attachments/assets/64fb5546-23ff-4623-95bb-fa1e13756842" />


3. Right-click **Users** → **New User**.

| Field | Value |
|-------|-------|
| Username | `ftpadm` |
| Description | User for FTP backup server |
| Password | `(set strong password)` |
| User must change password at next logon | ❌ Unchecked |
| User cannot change password | ✅ Checked |
| Password never expires | ✅ Checked |
| Account is disabled | ❌ Unchecked |

4. Click **Create**, then **Close**.

---

### Step 3 — Configure FTP Authentication

1. Open **IIS Manager** (Server Manager → Tools → Internet Information Services Manager).
2. In the left panel, select the server hostname (`BACKUPSRV`).
3. Double-click **FTP Authentication**.

<img width="736" height="380" alt="image" src="https://github.com/user-attachments/assets/4de6a42c-e812-409e-8859-ab29c8ccde83" />


4. Set **Anonymous Authentication** to **Enabled**.
5. Set **Basic Authentication** to **Enabled**.

### Step 4 — Configure FTP Authorization Rules

1. In IIS Manager, select `BACKUPSRV`.
2. Double-click **FTP Authorization Rules**.

<img width="664" height="343" alt="image" src="https://github.com/user-attachments/assets/c1f17645-6821-416d-9634-29ff730532fc" />


3. In the right panel, click **Add Allow Rule**.
4. Configure:
   - Select **Specific user** → enter `ftpadm`
   - Permissions: check both **Read** and **Write**
5. Click **OK**.

---

### Step 5 — Create the FTP Site

<img width="415" height="408" alt="image" src="https://github.com/user-attachments/assets/e1b07b55-7eca-4a78-a4c3-6b3427591fd1" />


1. In IIS Manager, right-click **Sites → Add FTP Site**.

**Site Information:**

<img width="582" height="443" alt="image" src="https://github.com/user-attachments/assets/8affa41d-bbe9-4c0c-8d98-244db751b126" />



| Field | Value |
|-------|-------|
| FTP Site Name | `FTP_server_backup` |
| Physical Path | `C:\inetpub\ftproot\vcenter_backup` |

> Create the directory first if it does not exist:
> ```cmd
> mkdir C:\inetpub\ftproot\vcenter_backup
> ```

<img width="491" height="373" alt="image" src="https://github.com/user-attachments/assets/08ed1d05-22b7-4e15-a767-dad8fa85ba18" />



**Binding and SSL Settings:**

| Field | Value |
|-------|-------|
| IP Address | `192.168.1.99` |
| Port | `21` |
| Start FTP site automatically | ✅ Yes |
| SSL | No SSL |

2. Click **Next**, configure Authentication:
   - Anonymous: **Enabled**
   - Basic: **Enabled**
   - Allow access to: **Specific user** → `ftpadm`
   - Permissions: **Read** and **Write**

3. Click **Finish**.

---

### Step 6 — Verify FTP Server with WinSCP

<img width="504" height="336" alt="image" src="https://github.com/user-attachments/assets/0b94184e-3e2d-4526-be64-a279070ee097" />


1. Open **WinSCP**.
2. Create a new session:
   - Protocol: FTP
   - Host: `192.168.1.99`
   - Port: `21`
   - Username: `ftpadm`
   - Password: *(your FTP password)*
3. Click **Login**.
4. Verify you can see the `vcenter_backup` directory.

---

## Part 2 — Configure vCenter Backup

### Step 1 — Access VMware Appliance Management

<img width="512" height="76" alt="image" src="https://github.com/user-attachments/assets/707b7700-f0b7-458b-aad3-532b5cb8726a" />


Open a browser and navigate to:
```
https://192.168.1.100:5480
```
Or:
```
https://vsystem.helpy.com:5480
```

<img width="463" height="261" alt="image" src="https://github.com/user-attachments/assets/b91f33a9-8cdc-41a9-9e4b-490074debd4e" />


Log in with:
- **Username:** `root`
- **Password:** *(VCSA root password)*

### Step 2 — Navigate to Backup

<img width="299" height="258" alt="image" src="https://github.com/user-attachments/assets/b0a02928-8555-427e-ac03-71acfe442713" />


1. In the Appliance Management interface, click the **Backup** tab.

### Step 3 — Configure a Backup Schedule

<img width="492" height="370" alt="image" src="https://github.com/user-attachments/assets/07328f47-e9ff-4b40-9042-4ea33e779466" />


1. Click **Add** to create a new scheduled backup.
2. Configure:

| Field | Value |
|-------|-------|
| Backup Location | `ftp://192.168.1.99:21/ftproot/vcenter_backup` |
| Username | `ftpadm` |
| Password | *(FTP user password)* |
| Encryption Password | *(optional, for encrypted backups)* |
| Backup Schedule | Daily |
| Time | `04:36 PM` (or your preferred time) |
| Retain backups | `3` (number of backup copies to keep) |

3. Click **Create** to save the schedule.

### Step 4 — Run a Manual Backup (Verification)

<img width="590" height="465" alt="image" src="https://github.com/user-attachments/assets/6a9ab8e8-27b6-4d81-a0cb-b86b8a8b76fe" />


1. On the Backup page, click **Backup Now**.
2. Enter the same FTP server details as above.
3. Click **Start**.
4. Monitor the progress bar until the backup completes.

<img width="695" height="107" alt="image" src="https://github.com/user-attachments/assets/08c99e1e-f3b9-46cd-9730-3fa04fd3c7bd" />


---

## Backup File Locations

<img width="706" height="113" alt="image" src="https://github.com/user-attachments/assets/faa71ff2-cb3d-46c4-92dd-7fcc9f972c49" />


Backup files are stored on the FTP server at:

```
C:\inetpub\ftproot\vcenter_backup\vCenter\sn_vsystem.helpy.com\
```

Example backup file names:
```
M_6.7.0.14000_20191223-161713_    ← Manual backup
M_6.7.0.14000_20200106-162305_    ← Manual backup
S_6.7.0.14000_20191223-165909_    ← Scheduled backup
```

File naming format: `[M=Manual/S=Scheduled]_[Version]_[YYYYMMDD-HHMMSS]`

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Backup fails immediately | Incorrect FTP path format | Ensure format is exactly `ftp://IP:21/folder` |
| Authentication error | Wrong FTP username/password | Verify `ftpadm` credentials in IIS Manager |
| FTP directory not found | Directory doesn't exist | Create `C:\inetpub\ftproot\vcenter_backup` manually |
| Backup never runs on schedule | Time mismatch | Verify NTP is synchronized on VCSA (check Appliance Management → Time) |
| WinSCP cannot connect | FTP service not running | Open IIS Manager and verify FTP site is started |
