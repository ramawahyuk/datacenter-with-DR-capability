# DR Runbook — vCenter Backup and Restore

This runbook provides step-by-step procedures for performing vCenter Server backup and full restore operations.

---

## Runbook Metadata

| Field | Value |
|-------|-------|
| Runbook ID | DR-001 |
| Feature | vCenter Backup & Restore |
| Technology | VMware Appliance Management + FTP |
| RTO | ~20–30 minutes |
| RPO | Last scheduled backup timestamp |
| Last Validated | See testing/alpha-test-results.md |

---

## Manual Backup Procedure

### Prerequisites

- [ ] Backup server (BACKUPSRV / 192.168.1.99) is running
- [ ] FTP service is active on BACKUPSRV
- [ ] VCSA is accessible at `https://vsystem.helpy.com:5480`
- [ ] FTP credentials available (`ftpadm`)

### Steps

1. Open browser → Navigate to `https://192.168.1.100:5480` or `https://vsystem.helpy.com:5480`
2. Log in as `root` with the VCSA root password.
3. Click the **Backup** tab.
4. Click **Backup Now**.
5. Enter backup parameters:
   ```
   Backup Location : ftp://192.168.1.99:21/ftproot/vcenter_backup
   Username        : ftpadm
   Password        : <ftp_password>
   ```
6. Click **Start**.
7. Monitor the progress bar until backup shows **Completed**.
8. Verify backup files on BACKUPSRV using WinSCP:
   - Connect to `192.168.1.99` via FTP
   - Navigate to `ftproot/vcenter_backup/vCenter/sn_vsystem.helpy.com/`
   - Confirm new backup folder with today's timestamp exists.

### Expected Backup File Format
```
M_6.7.0.14000_YYYYMMDD-HHMMSS_    ← Manual backup
S_6.7.0.14000_YYYYMMDD-HHMMSS_    ← Scheduled backup
```

---

## Configure Scheduled Backup

### Steps

1. Access VMware Appliance Management at `https://192.168.1.100:5480`.
2. Click **Backup** tab.
3. Click **Add** to create a schedule.
4. Configure:
   ```
   Backup Location : ftp://192.168.1.99:21/ftproot/vcenter_backup
   Username        : ftpadm
   Password        : <ftp_password>
   Schedule        : Daily
   Time            : 04:36 PM (adjust to your maintenance window)
   Retain          : 3 backups
   ```
5. Click **Create**.
6. Verify the schedule appears with **Enabled** status.

---

## Restore Procedure

> **Use this procedure when the VCSA VM is lost, deleted, or corrupted.**

### Prerequisites

- [ ] VCSA 6.7 ISO available (same version as backed-up VCSA)
- [ ] Access to backup files on BACKUPSRV
- [ ] Target ESXi host available (Sapcore5 / 192.168.1.28)
- [ ] FTP connectivity from installer machine to BACKUPSRV

### Steps

1. On the Windows client machine, extract the VCSA 6.7 ISO.
2. Navigate to `vcsa-ui-installer → win32`.
3. Run `installer.exe`.
4. Select **Restore** (not Install).
5. Click **Next**.

**Stage 1 — Connect to Backup**

6. Enter backup location:
   ```
   Backup Location : ftp://192.168.1.99:21/ftproot/vcenter_backup
   Username        : ftpadm
   Password        : <ftp_password>
   ```
7. Click **Connect**.
8. Browse and select the backup file to restore from.
9. Review backup metadata — confirm hostname and version.
10. Click **Next**.

**Stage 1 — Target Host**

11. Enter the target ESXi host:
    ```
    Host     : 192.168.1.28
    Port     : 443
    Username : root
    Password : <esxi_root_password>
    ```
12. Enter VM name and HTTPS port:
    ```
    VM Name    : VCSA-1
    HTTPS Port : 443
    ```
13. Select deployment size: **Tiny**
14. Select storage mode: **Thin Disk**
15. Select datastore available on Sapcore5.
16. Network settings auto-populate from backup data — verify they are correct.
17. Click **Finish** to begin Stage 1.

**Stage 2 — Configure Restored VCSA**

18. After Stage 1 completes, Stage 2 starts automatically.
19. Verify pre-populated settings from backup.
20. Click **Finish** to complete restore.

### Verification

After restore completes:

- [ ] Ping `192.168.1.100` — should reply
- [ ] Open `https://vsystem.helpy.com/ui` — should show vSphere Web Client login
- [ ] Log in with `administrator@vsphere.local`
- [ ] Verify all hosts, VMs, and clusters are visible
- [ ] Run a test backup to confirm the backup path still works

---

## Rollback

If restore fails:

1. Delete the partially deployed VCSA VM from the target ESXi host.
2. Select a different (older) backup file and retry from Step 6.
3. If all backups fail, escalate to VMware Support with backup file details.

---

## Contacts and Escalation

| Role | Responsibility |
|------|---------------|
| Infrastructure Admin | Perform backup/restore procedures |
| VMware Administrator | Troubleshoot VCSA issues |
| Network Admin | Verify FTP connectivity if backup fails |
