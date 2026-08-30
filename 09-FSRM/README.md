# 09 - File Server Resource Manager (FSRM)

## Purpose

This phase adds storage-governance controls to the departmental file server. It implements independent hard quotas, active image-file screening, and storage reports, then validates enforcement from both the server and a Windows client.

## Verified environment

| Component | Configuration |
| --- | --- |
| Server | `PC26` |
| Server operating system | Windows Server 2025 Standard Evaluation |
| Client | Windows 11 domain client |
| Department root | `F:\CompanyData\Departments` |
| Network share | `\\PC26\Departments` |
| Mapped drive used for testing | `S:` |
| Departments | Finance, HR, IT, Management, Marketing, Sales |

FSRM supplements SMB and NTFS authorization. It does not replace the AGDLP permission model implemented in Phase 08.

## Quota management

### Design

A `5 GB Limit` template was applied as a hard Auto Apply quota to `F:\CompanyData\Departments`. FSRM created an independent quota for each existing department folder and will apply the same template to new child folders.

```text
F:\CompanyData\Departments
├── Finance     → 5 GB hard quota
├── HR          → 5 GB hard quota
├── IT          → 5 GB hard quota
├── Management  → 5 GB hard quota
├── Marketing   → 5 GB hard quota
└── Sales       → 5 GB hard quota
```

This is different from placing one quota on the parent as a single shared limit. A hard quota prevents additional writes after the limit is reached; a soft quota would only monitor usage.

### Quota validation

The client-side copy test produced an insufficient-space result when the IT quota was effectively full. The server console then showed the IT folder at approximately 99 percent of its 5 GB limit while the other department quotas remained independent.

## File screening

### Configuration

| Setting | Value |
| --- | --- |
| Template | `Block Images` |
| Screening mode | Active |
| File group | Image Files |
| Applied path | `F:\CompanyData\Departments` |

Active screening blocks matching file types; passive screening would record or notify without preventing the write.

### File-screen validation

The same IT location accepted creation of a text document but rejected an attempted image copy. The Windows dialog is a generic access-denied message; paired with the verified active screen and the successful text-file test, it is consistent with the configured image restriction and is not presented as a distinct FSRM event-log capture.

## Storage reports

The captured report configuration includes:

| Report control | Value |
| --- | --- |
| Report types | Large Files; Least Recently Accessed Files |
| Large-file threshold | 50 MB |
| Least-recently-accessed threshold | 3 days |
| Output format | DHTML |
| Scope | Department storage beneath `F:\CompanyData\Departments` |

A generated Large Files report detected:

| Field | Reported value |
| --- | --- |
| File | `large_file_60mb.txt` |
| Folder | `F:\CompanyData\Departments\IT` |
| Owner | `VIREXON\S.ahmed` |
| Size on disk | 65.0 MB |
| Files detected | 1 |

The report console shows a task and a next-run time, but the captured properties do not expose a readable weekly recurrence or a Thursday/1:00 PM schedule. This document therefore verifies the report types, parameters, scope, output, and generated result without claiming an unsupported recurrence.

No automatic deletion, archiving, or file-management action was configured.

## Control separation

| Layer | Question answered |
| --- | --- |
| Share and NTFS permissions | Who can access or change the data? |
| FSRM quota | How much data may a department store? |
| FSRM file screen | What file types may be stored? |
| FSRM storage report | How is storage being consumed? |

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [FSRM Installation](Screenshots/01-FSRM-Role-Installation-Verification.png) | FSRM role service and management tools installed successfully. |
| 02 | [Auto Apply Quota Configuration](Screenshots/02-FSRM-Auto-Apply-Quota-Configuration.png) | Five-gigabyte hard template applied to existing and new subfolders. |
| 03 | [Auto Apply Quota Verification](Screenshots/03-FSRM-Auto-Apply-Quota-Verification.png) | Independent department quotas and the Auto Apply source entry. |
| 04 | [Client Quota Enforcement](Screenshots/04-Quota-Limit-Client-Verification.png) | Additional client data cannot be written at the limit. |
| 05 | [Server Quota Usage](Screenshots/05-Quota-Usage-Server-Verification.png) | IT usage near the 5 GB limit and independent department entries. |
| 06 | [File Screen Template](Screenshots/06-File-Screen-Template-Configuration.png) | Active `Block Images` template using the Image Files group. |
| 07 | [Applied File Screen](Screenshots/07-File-Screen-Configuration.png) | Template applied to the department root. |
| 08 | [Client File-Screen Test](Screenshots/08-File-Screening-Client-Verification.png) | Text allowed and image copy rejected in the same location. |
| 09 | [Report Configuration](Screenshots/09-Storage-Reports-Configuration.png) | Selected report types, parameters, DHTML output, and report task. |
| 10 | [Large Files Summary](Screenshots/10-Storage-Report-Large-Files-Summary.png) | One 65 MB file attributed to `VIREXON\S.ahmed`. |
| 11 | [Large File Details](Screenshots/11-Storage-Report-Large-File-Details.png) | Exact filename, path, owner, size, and last-access value. |

## Outcome

FSRM is operational as a governance layer over the departmental share. Each department has its own enforced 5 GB limit, image files are actively screened while normal text data remains writable, and storage reporting identifies large and inactive data using verified parameters. Claims about report scheduling are deliberately limited to what the screenshots display.
