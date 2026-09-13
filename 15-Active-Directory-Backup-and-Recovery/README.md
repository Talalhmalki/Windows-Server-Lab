# 15 - Active Directory Backup and Recovery

## Overview

This phase documents a System State backup and non-authoritative recovery of `PC27` in the `virexon.local` Active Directory environment.

The backup was created with Windows Server Backup and stored on a dedicated local volume. Its version and available recovery types were verified before restoring the same Domain Controller in Directory Services Restore Mode (DSRM).

A test Organizational Unit (OU) was created after the backup to validate the restored server's ability to receive newer directory data from `PC26`.

After recovery, the required services were running, the `SYSVOL` and `NETLOGON` shares were available, and the test OU was present on `PC27`. The final replication summary reported zero failures for both Domain Controllers.

| Item | Details |
|---|---|
| Recovery Test Status | Completed |
| Active Directory Domain | `virexon.local` |
| Active Directory Site | `Riyadh-HQ` |
| Backup and Recovery Target | `PC27.virexon.local` |
| Available Replication Partner | `PC26.virexon.local` |
| Backup Scope | System State |
| Backup Destination | `AD-Recovery (B:)` |
| Recovery Method | Non-authoritative System State restore in DSRM |
| Evidence | 9 screenshots |

## Objectives

- Establish a replication baseline before taking the backup.
- Create and verify a System State backup of `PC27`.
- Introduce a directory change after the backup.
- Restore `PC27` from its own backup using Windows Server Backup.
- Verify essential Domain Controller services and shared folders after recovery.
- Confirm that the restored server received the newer directory change.
- Record the final replication status of both Domain Controllers.

## Lab Environment

| System | IPv4 Address | Role in This Phase |
|---|---|---|
| `PC26.virexon.local` | `192.168.1.2` | Existing writable Domain Controller, DNS server, and replication partner |
| `PC27.virexon.local` | `192.168.1.3` | Additional writable Domain Controller, DNS server, Global Catalog, and backup/recovery target |

| Component | Configuration |
|---|---|
| Virtualization Platform | VMware Workstation Pro |
| Network | Host-only lab network |
| Recovery Target Operating System | Windows Server 2025 |
| Backup Storage | Separate 60 GB NTFS volume assigned to `B:` |
| Backup Tool | Windows Server Backup |
| Directory Management | Active Directory Users and Computers |
| Verification Tools | `wbadmin`, `dsquery`, `sc`, `net share`, and `repadmin` |

`PC26` remained available as the replication partner while `PC27` underwent recovery. The established FSMO role placement remained on `PC26`.

## Recovery Design

System State backup protects the Active Directory database and associated system components, including SYSVOL, the registry, and boot-related files.

The recovery exercise used a **non-authoritative restore**. This restores the local directory data from the selected backup and allows the recovered Domain Controller to receive subsequent directory updates from its replication partners.

The test OU, `AD-Recovery-Test`, was created after the backup completed. Its presence on `PC27` after recovery provided evidence that the restored server received directory data newer than the backup.

| Stage | Test Design |
|---|---|
| Backup | Capture the System State of `PC27` before creating the test OU |
| Directory Change | Create `AD-Recovery-Test` on `PC26` and verify its presence on `PC27` |
| Recovery | Restore the earlier System State backup to the same `PC27` |
| Verification | Confirm that the test OU is present on the recovered server and check replication health |

This exercise covered recovery of an existing Domain Controller with an operational replication partner available.

---

## 01 - Pre-Recovery AD Replication Health

Replication health was checked before creating the System State backup:

```cmd
repadmin /replsummary
```

Both Domain Controllers appeared as replication sources and destinations. The retained baseline showed zero replication failures.

| Domain Controller | Source Failures | Destination Failures |
|---|---|---|
| `PC26` | `0 / 5` | `0 / 5` |
| `PC27` | `0 / 5` | `0 / 5` |

This established the replication baseline for the backup and recovery exercise.

![Pre-Recovery AD Replication Health](Screenshots/01-Pre-Recovery-AD-Replication-Health.png)

---

## 02 - System State Backup Configuration

Windows Server Backup was used to configure a one-time backup on `PC27`.

The backup included **System State** and used the dedicated `AD-Recovery (B:)` volume as its destination.

| Setting | Selected Value |
|---|---|
| Backup Operation | Backup Once |
| Backup Options | Different options |
| Backup Configuration | Custom |
| Selected Backup Item | System State |
| Destination Type | Local drives |
| Backup Destination | `AD-Recovery (B:)` |
| VSS Setting | VSS Copy Backup |
| File Exclusions | None |

The confirmation screen verified the selected backup scope, destination, and VSS setting before the operation started.

![System State Backup Configuration](Screenshots/02-System-State-Backup-Configuration.png)

---

## 03 - System State Backup Completion

The System State backup completed successfully.

| Verification Item | Observed Result |
|---|---|
| Backup Item | System State |
| Backup Destination | `B:` |
| Operation Status | Completed |
| Windows Server Backup Result | Successful |
| Data Transferred | `12.25 GB` |

The completion result confirmed that Windows Server Backup finished writing the selected System State data to the backup destination.

![System State Backup Completion](Screenshots/03-System-State-Backup-Completion.png)

---

## 04 - System State Backup Version Verification

The available backup was queried from an elevated Command Prompt:

```cmd
wbadmin get versions -backupTarget:B: -machine:PC27
```

The command identified the backup stored on `B:` for `PC27` and listed its available recovery types.

| Backup Detail | Recorded Value |
|---|---|
| Backup Time Displayed | September 8, 2026, at 4:36 PM |
| Backup Target | `B:` |
| Backup Machine Selected | `PC27` |
| Version Identifier | `09/08/2026-13:36` |
| Available Recovery Types | Volumes, Files, Applications, and System State |

The version identifier was recorded exactly as returned by `wbadmin`. The Recovery Wizard used the corresponding backup date and time displayed in the graphical interface.

**Result:** The backup was discoverable, and System State was listed as an available recovery type.

![System State Backup Version Verification](Screenshots/04-System-State-Backup-Version-Verification.png)

---

## 05 - Post-Backup Directory Change Verification

After the backup completed, an empty test OU named `AD-Recovery-Test` was created at the domain root on `PC26`.

The OU was then queried directly from `PC27`:

```cmd
dsquery * "OU=AD-Recovery-Test,DC=virexon,DC=local" -s PC27 -scope base -attr name whenCreated
```

| Attribute | Observed Value |
|---|---|
| `name` | `AD-Recovery-Test` |
| `whenCreated` | `09/08/2026 17:32:08` |
| Queried Domain Controller | `PC27` |

The `-s PC27` parameter directed the query to the recovery target, confirming that the test OU had reached its directory replica before recovery.

The creation timestamp was retained exactly as displayed by the query for comparison after the restore.

**Result:** A directory change created after the backup was confirmed on `PC27`, establishing the recovery test condition.

![Post-Backup Directory Change Verification](Screenshots/05-Post-Backup-Directory-Change-Verification.png)

---

## 06 - DSRM Non-Authoritative System State Recovery

`PC27` was restarted in **Directory Services Restore Mode (DSRM)** to perform the System State recovery with Active Directory Domain Services offline.

DSRM was selected through:

**System Configuration (`msconfig`) → Boot → Safe boot → Active Directory repair**

The server was accessed using the local DSRM Administrator account and its DSRM password.

Windows Server Backup was then used to restore the selected backup.

| Recovery Setting | Selected Value |
|---|---|
| Recovery Tool | Windows Server Backup |
| Backup Selection | This server |
| Backup Date | September 8, 2026 |
| Backup Time Displayed | 4:36 PM |
| Recovery Type | System State |
| Recovery Destination | Original location |
| Perform an authoritative restore of Active Directory files | Unchecked |
| Automatically reboot the server | Unchecked |

The authoritative restore option remained unselected so that `PC27` could receive newer replicated data after returning to normal operation.

Windows Server Backup reported **Completed** and indicated that a restart was required to finish the recovery process.

After capturing the completion result, the Safe boot setting was cleared and `PC27` was restarted in normal mode. Verification continued after signing in with the domain administrative account.

![DSRM Non-Authoritative System State Recovery](Screenshots/06-DSRM-Non-Authoritative-System-State-Recovery.png)

---

## 07 - Post-Recovery DC Service Verification

After the server returned to normal operation, its identity and essential services were checked:

```cmd
hostname
sc query NTDS | findstr STATE
sc query DNS | findstr STATE
sc query Netlogon | findstr STATE
sc query DFSR | findstr STATE
net share
```

The hostname result confirmed that the checks were performed on `PC27`.

| Service | Purpose | Observed State |
|---|---|---|
| `NTDS` | Provides Active Directory Domain Services | `4 RUNNING` |
| `DNS` | Provides DNS service on the Domain Controller | `4 RUNNING` |
| `Netlogon` | Supports domain authentication and Domain Controller registration | `4 RUNNING` |
| `DFSR` | Provides DFS Replication, including SYSVOL replication | `4 RUNNING` |

The shared-folder check also confirmed the following:

| Share | Reported Location |
|---|---|
| `SYSVOL` | `C:\WINDOWS\SYSVOL\sysvol` |
| `NETLOGON` | `C:\WINDOWS\SYSVOL\sysvol\virexon.local\SCRIPTS` |

These checks established that the required services were running and the expected shares were published. Directory data and replication were verified in the following steps.

![Post-Recovery DC Service Verification](Screenshots/07-Post-Recovery-DC-Service-Verification.png)

---

## 08 - Post-Recovery Directory Replication Verification

The test OU was queried again directly from the recovered `PC27`:

```cmd
dsquery * "OU=AD-Recovery-Test,DC=virexon,DC=local" -s PC27 -scope base -attr name whenCreated
```

The query returned the same OU name and creation timestamp recorded before recovery.

| Attribute | Before Recovery | After Recovery |
|---|---|---|
| `name` | `AD-Recovery-Test` | `AD-Recovery-Test` |
| `whenCreated` | `09/08/2026 17:32:08` | `09/08/2026 17:32:08` |
| Queried Domain Controller | `PC27` | `PC27` |

Because this OU was created after the backup, its presence on the recovered server demonstrated receipt of the newer directory change from `PC26`.

**Result:** The restored Domain Controller received the test directory object through replication, matching the expected behavior of a non-authoritative restore.

![Post-Recovery Directory Replication Verification](Screenshots/08-Post-Recovery-Directory-Replication-Verification.png)

---

## 09 - Final AD Replication Health

The final replication summary was captured after recovery and directory verification:

```cmd
repadmin /replsummary
```

The retained result, captured on September 9, 2026, showed both Domain Controllers with zero failures as replication sources and destinations.

| Domain Controller | Source Failures | Destination Failures | Reported Failure Percentage |
|---|---|---|---|
| `PC26` | `0 / 5` | `0 / 5` | `0%` |
| `PC27` | `0 / 5` | `0 / 5` | `0%` |

No replication error was displayed in the final summary.

**Result:** The final replication check met the test's success criterion of zero reported failures for both Domain Controllers.

![Final AD Replication Health](Screenshots/09-Final-AD-Replication-Health.png)

---

## Validation Summary

| Evidence | Validation | Observed Result |
|---|---|---|
| 01 | Replication baseline | Zero failures reported for both Domain Controllers |
| 02 | Backup configuration | System State selected with `B:` as the destination |
| 03 | Backup completion | Successful backup with `12.25 GB` transferred |
| 04 | Backup version verification | Backup identified and System State recovery listed |
| 05 | Post-backup directory change | Test OU confirmed on `PC27` before recovery |
| 06 | System State recovery | Recovery operation reported Completed |
| 07 | Services and shared folders | Required services running; `SYSVOL` and `NETLOGON` available |
| 08 | Directory verification after recovery | Test OU present on the recovered `PC27` |
| 09 | Final replication check | Zero failures reported in both directions |

## Outcome

`PC27` was backed up and restored through a complete System State recovery exercise using Windows Server Backup.

The evidence records the backup configuration, successful backup creation, version verification, recovery in DSRM, and verification after returning to normal operation.

The post-backup test OU was present on the recovered server, the required services and shares were available, and the final replication summary reported zero failures between `PC26` and `PC27`.
