# 10 - Shadow Copies

## Purpose

This phase implements and validates Shadow Copies for the departmental file server. The objective is to give users a fast, client-accessible way to inspect and restore an earlier folder state without starting a full backup recovery.

## Verified environment

| Component | Configuration |
| --- | --- |
| Server | `PC26.virexon.local` |
| Server operating system | Windows Server 2025 Standard Evaluation |
| Server IP address | `192.168.1.2` |
| Client | `PC-IT-01` |
| Client operating system | Windows 11 |
| Protected volume | `F:` |
| Department data | `F:\CompanyData\Departments` |
| Network share | `\\PC26\Departments` |
| Client drive used for testing | `S:` |
| Shadow Copy storage volume | `G:` |

Shadow Copies complement the authorization and storage-governance controls established in Phases 08 and 09. They do not replace an independent backup.

## Recovery design

The implementation protects `F:` at the volume level because Shadow Copies are not configured for an individual folder. The storage area for those snapshots is `G:`, with a configured maximum of `39,936 MB`.

```text
F:  Department data and protected volume
G:  Shadow Copy storage for F:
```

Although `F:` and `G:` are separate volumes, both reside on the same underlying virtual disk in this lab. This arrangement separates allocation but does not protect against loss of that virtual disk or its host.

## Schedule

Two recovery points were scheduled on each working day:

| Days | First recovery point | Second recovery point |
| --- | ---: | ---: |
| Sunday through Thursday | `7:00 AM` | `12:00 PM` |

Friday and Saturday are not included in the captured schedule.

## Server-side validation

After the configuration was completed, a recovery point was created for controlled testing. The captured Shadow Copies console shows:

| Observation | Verified value |
| --- | --- |
| Protected volume | `F:` |
| Recovery-point timestamp | `8/18/2026 8:58 PM` |
| Storage location | `G:` |
| Storage in use at capture time | Approximately `1.88 GB` |

These observations verify that a recovery point existed and that `G:` was being used as configured.

## Client recovery test

The recovery workflow was validated from `PC-IT-01` against the IT department folder on `S:`.

| State | Visible content |
| --- | --- |
| Earlier folder state | `VIREXON 1` |
| Current folder state before restoration | `VIREXON 2` |

The Previous Versions interface exposed the `8/18/2026 8:58 PM` recovery point. Opening it showed `VIREXON 1` while the live folder showed `VIREXON 2`, confirming that the two states were distinct.

The earlier folder version was then restored. Windows reported that the folder had been successfully restored, and `VIREXON 1` was visible again in the live location.

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [Storage Configuration](Screenshots/01-Shadow-Copies-Storage-Configuration.png) | `F:` is protected, `G:` is the storage area, and the maximum allocation is `39,936 MB`. |
| 02 | [Schedule Configuration](Screenshots/02-Shadow-Copies-Schedule-Configuration.png) | Recovery points are scheduled at `7:00 AM` and `12:00 PM` from Sunday through Thursday. |
| 03 | [Recovery-Point Verification](Screenshots/03-Shadow-Copy-Creation-Verification.png) | The `8/18/2026 8:58 PM` recovery point exists and storage is in use on `G:`. |
| 04 | [Client Previous Versions](Screenshots/04-Previous-Versions-Client-Verification.png) | The recovery point is available to the Windows client through Previous Versions. |
| 05 | [Content Comparison](Screenshots/05-Previous-Version-Content-Verification.png) | The earlier `VIREXON 1` state differs from the current `VIREXON 2` state. |
| 06 | [Restore Verification](Screenshots/06-Previous-Version-Restore-Verification.png) | Windows reports a successful restore and the earlier content is present again. |

## Operational considerations

- Shadow Copies provide rapid recovery from common file and folder changes.
- Storage consumption and retention depend on change rate and the configured maximum allocation.
- Older recovery points can be removed as the storage area fills.
- Client-accessible Previous Versions reduce routine restore effort, but access remains subject to the existing SMB and NTFS permissions.
- Because `F:` and `G:` share one underlying virtual disk, Windows Server Backup in Phase 11 supplies the separate recovery layer.

## Outcome

Shadow Copies are enabled for the departmental data volume, stored on `G:`, and scheduled twice per working day. The server-side recovery point, client-visible previous version, content difference, and successful restoration are all represented in the captured evidence.

**10 - Shadow Copies — Completed ✅**
