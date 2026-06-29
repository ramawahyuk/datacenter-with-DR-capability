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
6. Check **Web Server → FTP Server → FTP Service**.
7. Click **Next** through Features, then **Install**.
8. Wait for installation to complete.

### Step 2 — Create FTP User

1. Open **Computer Management** (right-click Start → Computer Management).
2. Navigate to **Local Users and Groups → Users**.
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

### Step 3 — Configure FTP Authentication

1. Open **IIS Manager** (Server Manager → Tools → Internet Information Services Manager).
2. In the left panel, select the server hostname (`BACKUPSRV`).
3. Double-click **FTP Authentication**.
4. Set **Anonymous Authentication** to **Enabled**.
5. Set **Basic Authentication** to **Enabled**.

### Step 4 — Configure FTP Authorization Rules

1. In IIS Manager, select `BACKUPSRV`.
2. Double-click **FTP Authorization Rules**.
3. In the right panel, click **Add Allow Rule**.
4. Configure:
   - Select **Specific user** → enter `ftpadm`
   - Permissions: check both **Read** and **Write**
5. Click **OK**.

### Step 5 — Create the FTP Site

1. In IIS Manager, right-click **Sites → Add FTP Site**.

**Site Information:**

| Field | Value |
|-------|-------|
| FTP Site Name | `FTP_server_backup` |
| Physical Path | `C:\inetpub\ftproot\vcenter_backup` |

> Create the directory first if it does not exist:
> ```cmd
> mkdir C:\inetpub\ftproot\vcenter_backup
> ```

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

### Step 6 — Verify FTP Server with WinSCP

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

Open a browser and navigate to:
```
https://192.168.1.100:5480
```
Or:
```
https://vsystem.helpy.com:5480
```

Log in with:
- **Username:** `root`
- **Password:** *(VCSA root password)*

### Step 2 — Navigate to Backup

1. In the Appliance Management interface, click the **Backup** tab.

### Step 3 — Configure a Backup Schedule

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

1. On the Backup page, click **Backup Now**.
2. Enter the same FTP server details as above.
3. Click **Start**.
4. Monitor the progress bar until the backup completes.

---

## Backup File Locations

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
