# 11 - Windows Server Backup

## Purpose

This phase implements and validates Windows Server Backup (WSB) for the departmental file server. It adds a recovery layer stored on a separate virtual disk, verifies a completed backup, and confirms file-level recovery through the WSB Recovery Wizard.

## Verified environment

| Component | Configuration |
| --- | --- |
| Server | `PC26` |
| Server operating system | Windows Server 2025 Standard Evaluation |
| Protected data | `F:\CompanyData\Departments` |
| Network presentation of the data | `\\PC26\Departments` |
| Shadow Copy storage | `G:` |
| Backup feature | Windows Server Backup |
| Backup destination | Dedicated 60 GB virtual disk |
| Hypervisor | VMware Workstation Pro |
| Lab network | Host-only |

WSB protects the local data path, not the SMB path through which clients consume the same data.

## Protection architecture

```text
PC26
├── System virtual disk
│   └── C:  Windows Server
├── Data virtual disk
│   ├── F:  Department data
│   └── G:  Shadow Copy storage
└── Dedicated 60 GB virtual disk
    └── Windows Server Backup destination
```

The dedicated destination is separate from the virtual disk that contains `F:` and `G:`. This improves recovery isolation within the lab, but it is still attached to the same virtual machine and hosted on the same VMware infrastructure. It is not an off-host or offsite backup.

## Backup preparation

### Dedicated destination

A new 60 GB virtual disk was added for WSB. The initial disk view shows it as a separate, unallocated disk rather than another partition on the production data disk. WSB later selected that disk as its dedicated destination.

### Feature installation

The Windows Server Backup feature and its management tools were installed successfully on `PC26`.

## Backup configuration

| Setting | Verified configuration |
| --- | --- |
| Backup scope | `F:\CompanyData\Departments` |
| Destination | Dedicated 60 GB virtual disk |
| Schedule | Daily at `12:00 AM` and `12:00 PM` |
| Approximate interval | 12 hours |
| Excluded files | None shown |
| VSS option | VSS Copy Backup |

Limiting the selection to the department root aligns the backup scope with the documented business data rather than expanding it to the entire server.

## Backup verification

The WSB console records a successful backup with the following visible details:

| Observation | Captured value |
| --- | --- |
| Completion time | `12:34 AM` |
| Status | Successful |
| Copies available | 1 |
| Next scheduled backup | `12:00 PM` |
| Destination capacity shown | `59.86 GB` |
| VSS setting | VSS Copy Backup |

The completion capture verifies that WSB produced a usable backup and retained the twice-daily schedule. It does not identify whether that specific run was initiated by the schedule or by an administrator, so no trigger method is asserted here.

## Recovery validation

The WSB Recovery Wizard was used to browse the stored department hierarchy and select `Test Folder`, which contained two test files. The recovery operation then reached `Completed` status, with both files represented in the recovery result.

This verifies two separate requirements:

1. The selected department data was present in the backup catalog.
2. WSB could process the selected data through file-level recovery.

The screenshots demonstrate successful selection and recovery. They do not independently document every preparation step of the test scenario, so the validation claim is intentionally limited to the captured evidence.

## Recovery-layer comparison

| Characteristic | Shadow Copies | Windows Server Backup |
| --- | --- | --- |
| Primary use | Fast previous-version access | Backup-based recovery |
| Protected scope in this lab | `F:` volume snapshots | `F:\CompanyData\Departments` |
| Storage | `G:` | Dedicated 60 GB virtual disk |
| Underlying-disk separation from `F:` | No | Yes |
| Client Previous Versions integration | Yes | No |
| Recovery interface | Previous Versions | WSB Recovery Wizard |
| Recovery validated | Yes | Yes |

The two mechanisms are complementary. Shadow Copies optimize routine version recovery; WSB provides a separate backup repository and recovery workflow.

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [Dedicated Backup Disk](Screenshots/01-WSB-Dedicated-Backup-Disk.png) | A separate 60 GB virtual disk is available for backup use. |
| 02 | [Feature Installation](Screenshots/02-WSB-Feature-Installation-Verification.png) | Windows Server Backup and its management tools installed successfully. |
| 03 | [Backup Scope](Screenshots/03-Backup-Scope-Configuration.png) | `F:\CompanyData\Departments` is the selected backup content. |
| 04 | [Backup Schedule](Screenshots/04-Backup-Schedule-Configuration.png) | Daily runs are configured for `12:00 AM` and `12:00 PM`. |
| 05 | [Backup Destination](Screenshots/05-Backup-Destination-Configuration.png) | The separate 60 GB disk is selected as the WSB destination. |
| 06 | [Backup Completion](Screenshots/06-Backup-Completion-Verification.png) | A backup completed successfully and the next scheduled time is visible. |
| 07 | [Recovery Selection](Screenshots/07-File-Recovery-Configuration.png) | `Test Folder` and its two files are available for selection in the backup. |
| 08 | [Recovery Completion](Screenshots/08-File-Recovery-Verification.png) | The selected file-level recovery completed successfully. |

## Operational considerations

- A successful backup status should be paired with periodic recovery testing.
- The twice-daily schedule limits the nominal interval between backup opportunities; it does not guarantee a recovery point unless a run completes successfully.
- Capacity, backup age, and job results should be monitored as the protected data grows.
- The dedicated virtual disk is a stronger boundary than the Shadow Copy arrangement, but host or datastore failure could still affect both production and backup disks.
- A production design should add off-host or offsite copies, retention requirements, encryption, monitoring, and documented recovery objectives.

## Outcome

Windows Server Backup protects the departmental data on a dedicated 60 GB virtual disk, with daily runs scheduled at midnight and noon. The evidence confirms the configured scope and destination, a successful backup, recoverable test content, and a completed file-level recovery.

**11 - Windows Server Backup — Completed ✅**
