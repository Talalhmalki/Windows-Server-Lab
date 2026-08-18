# 10 - Shadow Copies

## Overview

This section documents the implementation and verification of **Shadow Copies** on the VIREXON file server.

The objective was to provide a fast recovery method for departmental shared data by allowing users to access and restore previous versions of files and folders without requiring a full backup restoration.

Shadow Copies were configured on the volume hosting the company data and were tested directly from the Windows 11 domain client.

The implementation followed this workflow:

**Configure → Schedule → Create Shadow Copy → Modify Data → Access Previous Version → Restore → Verify**

---

## Environment

| Component | Configuration |
|---|---|
| Domain | `VIREXON.LOCAL` |
| Server | `PC26.virexon.local` |
| Server IP | `192.168.1.2` |
| Server OS | Windows Server 2022 |
| Client | `PC-IT-01` |
| Client OS | Windows 11 |
| Company Data Volume | `F:` |
| Company Data Path | `F:\CompanyData\Departments` |
| Department Share | `\\PC26\Departments` |
| Mapped Drive | `S:` |
| Shadow Copy Storage Volume | `G:` |

---

## Business Requirement

The departmental file server contains shared company data that may be modified, renamed, or deleted by users during normal daily work.

The requirement was to provide a quick recovery method that allows previous versions of shared files and folders to be accessed without performing a full server backup restore.

Shadow Copies were therefore implemented for the volume hosting the departmental data.

---

## Shadow Copy Storage Configuration

The company departmental data is stored on:

```text
F:\CompanyData\Departments
```

Because Shadow Copies are configured at the **volume level**, the protected volume is:

```text
F:
```

A separate volume was created for Shadow Copy storage:

```text
G:
```

The final storage design was:

```text
F:  → Company Data
G:  → Shadow Copy Storage for F:
```

The Shadow Copy settings for `F:` were configured to use `G:` as the storage area.

The configured maximum storage size was:

```text
39,936 MB
```

This keeps Shadow Copy storage separate from the main `F:` data volume.

> `F:` and `G:` are located on the same underlying virtual disk, so this configuration provides storage separation but does not replace a proper backup solution.

### Storage Configuration Evidence

![Shadow Copies Storage Configuration](Screenshots/01-Shadow-Copies-Storage-Configuration.png)

---

## Shadow Copy Schedule

A scheduled recovery design was configured based on the working week used in the lab environment.

Shadow Copies are scheduled on:

- Sunday
- Monday
- Tuesday
- Wednesday
- Thursday

Two Shadow Copies are scheduled during each working day:

| Time | Days |
|---|---|
| `7:00 AM` | Sunday - Thursday |
| `12:00 PM` | Sunday - Thursday |

Friday and Saturday are excluded from the schedule.

This provides two recovery points during each normal working day.

### Schedule Configuration Evidence

![Shadow Copies Schedule Configuration](Screenshots/02-Shadow-Copies-Schedule-Configuration.png)

---

## Shadow Copy Creation

After completing the storage and schedule configuration, Shadow Copies were enabled on:

```text
F:
```

A manual Shadow Copy was then created to perform a controlled recovery test.

The created recovery point appeared with the timestamp:

```text
8/18/2026 8:58 PM
```

The server also showed Shadow Copy storage being used on:

```text
G:
```

with approximately:

```text
1.88 GB
```

in use during the verification.

This confirmed that the Shadow Copy was successfully created and that `G:` was being used as the configured storage location.

### Shadow Copy Creation Evidence

![Shadow Copy Creation Verification](Screenshots/03-Shadow-Copy-Creation-Verification.png)

---

## Client Recovery Test

The recovery test was performed from the domain client:

```text
PC-IT-01
```

The test location was the IT departmental folder through the mapped drive:

```text
S:\IT
```

Before the Shadow Copy was created, the test file existed as:

```text
VIREXON 1
```

After the Shadow Copy was created, the live file was modified and renamed to:

```text
VIREXON 2
```

This created a visible difference between the current data and the data stored inside the Shadow Copy.

---

## Previous Versions Verification

From `PC-IT-01`, the **Previous Versions** tab was opened for the `IT` departmental folder.

A previous version was displayed with the timestamp:

```text
8/18/2026 8:58 PM
```

This matched the Shadow Copy that had been manually created on the server.

This confirmed that the recovery point created on `PC26` was available from the client through the shared departmental folder.

### Client Verification Evidence

![Previous Versions Client Verification](Screenshots/04-Previous-Versions-Client-Verification.png)

---

## Previous Version Content Verification

Before restoring the previous version, it was opened to verify its contents.

The current `IT` folder contained:

```text
VIREXON 2
```

The previous version of the same folder contained:

```text
VIREXON 1
```

This confirmed that the Shadow Copy preserved the earlier state of the departmental folder before the file was modified and renamed.

The previous version was inspected before restoration to ensure that the correct recovery point had been selected.

### Previous Version Content Evidence

![Previous Version Content Verification](Screenshots/05-Previous-Version-Content-Verification.png)

---

## Previous Version Restore

The previous version of the `IT` folder was restored from `PC-IT-01`.

Windows confirmed the restore operation with the message:

```text
The folder has been successfully restored to the previous version.
```

After the restore completed, the earlier file:

```text
VIREXON 1
```

was visible again in the current departmental folder.

This confirmed that the Previous Versions recovery process worked successfully from the Windows 11 client.

### Restore Verification Evidence

![Previous Version Restore Verification](Screenshots/06-Previous-Version-Restore-Verification.png)

---

## Verification Summary

| Test | Result |
|---|---|
| Correct company data volume identified as `F:` | ✅ Passed |
| Shadow Copy storage configured on `G:` | ✅ Passed |
| Maximum Shadow Copy storage configured | ✅ Passed |
| Sunday - Thursday schedule configured | ✅ Passed |
| 7:00 AM schedule configured | ✅ Passed |
| 12:00 PM schedule configured | ✅ Passed |
| Shadow Copies enabled on `F:` | ✅ Passed |
| Manual Shadow Copy created successfully | ✅ Passed |
| Shadow Copy storage used on `G:` | ✅ Passed |
| Previous Version visible from `PC-IT-01` | ✅ Passed |
| Previous folder contents accessible | ✅ Passed |
| Earlier file state visible | ✅ Passed |
| Previous Version restored successfully | ✅ Passed |
| Restored file visible in the live folder | ✅ Passed |

---

## Screenshot Documentation

| # | Screenshot | Evidence |
|---|---|---|
| 01 | `01-Shadow-Copies-Storage-Configuration.png` | Shows `F:` using `G:` as the Shadow Copy storage volume |
| 02 | `02-Shadow-Copies-Schedule-Configuration.png` | Shows the two scheduled recovery points for Sunday - Thursday |
| 03 | `03-Shadow-Copy-Creation-Verification.png` | Confirms successful Shadow Copy creation and storage usage on `G:` |
| 04 | `04-Previous-Versions-Client-Verification.png` | Confirms that the previous version is visible from `PC-IT-01` |
| 05 | `05-Previous-Version-Content-Verification.png` | Shows the difference between the live folder and the previous folder state |
| 06 | `06-Previous-Version-Restore-Verification.png` | Confirms successful restoration of the previous folder version |

---

## Key Takeaways

The Shadow Copies implementation demonstrated the following practical Windows Server concepts:

- Shadow Copies are configured at the **volume level**.
- `F:` is the protected company data volume.
- `G:` is used as the Shadow Copy storage volume.
- Recovery points can be created automatically through a schedule.
- Recovery points can also be created manually for testing or administrative purposes.
- Previous Versions can be accessed directly from a Windows client.
- Previous folder states can be inspected before performing a restore.
- Files that have been renamed or changed can be recovered from an earlier folder version.
- Shadow Copies provide fast recovery from common user mistakes.
- Shadow Copies are **not a replacement for backup**.

---

## Conclusion

Shadow Copies were successfully configured and verified for the departmental file server.

The company data volume `F:` was configured as the protected volume, while the dedicated `G:` volume was used for Shadow Copy storage.

Two scheduled recovery points were configured for every working day from Sunday through Thursday at `7:00 AM` and `12:00 PM`.

A manual Shadow Copy was created and successfully detected from the Windows 11 client through **Previous Versions**.

The recovery test confirmed that the client could:

1. Detect the previous version.
2. Open the previous folder state.
3. View the earlier `VIREXON 1` file.
4. Restore the previous version.
5. Verify that the earlier file was available again after restoration.

This implementation provides the VIREXON file server with a practical and efficient method for recovering previous versions of shared departmental data.

**10 - Shadow Copies — Completed ✅**
