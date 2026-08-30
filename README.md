# Windows Server Infrastructure Lab — VIREXON

A screenshot-backed Windows Server lab that demonstrates centralized identity, policy, networking, and file-services administration for the fictitious **VIREXON** organization. The environment is built in VMware Workstation and uses an isolated, single-server design for hands-on learning and repeatable validation.

## Environment

| Component | Implementation |
| --- | --- |
| Organization | VIREXON (fictitious) |
| Active Directory domain | `virexon.local` |
| Server | `PC26` — domain controller, DNS, DHCP, and file services |
| Server address | `192.168.1.2/24` |
| Server operating system | Windows Server 2025 Standard Evaluation, as shown in the Phase 01 system evidence |
| Client | `PC-IT-01` — Windows 11 Pro domain member |
| Virtualization | VMware Workstation Pro |
| Lab network | VMnet1 host-only network, `192.168.1.0/24` |
| Department share | `\\PC26\Departments` backed by `F:\CompanyData\Departments` |

## What the project demonstrates

- Active Directory Domain Services deployment and domain-controller validation.
- Department-based OU, user, computer, and security-group administration.
- Group Policy scoping with both positive and negative client-side validation.
- DNS forward and reverse lookup configuration and testing.
- DHCP scope, exclusion, reservation, and client lease validation.
- Department file shares using AGDLP, share permissions, and NTFS permissions.
- File Server Resource Manager quotas, file screening, and reporting.
- Shadow Copies configuration and previous-version recovery testing.
- Scheduled Windows Server Backup with dedicated-disk storage and file-recovery validation.

## Project phases

| Phase | Area | Evidence |
| --- | --- | --- |
| [01](01-Server-Preparation) | Server baseline and environment verification | 6 screenshots |
| [02](02-Active-Directory) | Active Directory Domain Services | 5 screenshots |
| [03](03-Organizational-Units) | Organizational Unit design | 4 screenshots |
| [04](04-Users-and-Groups) | Users, groups, and domain client | 11 screenshots |
| [05](05-Group-Policy) | Group Policy implementation and validation | 46 screenshots |
| [06](06-DNS) | DNS configuration and validation | 16 screenshots |
| [07](07-DHCP) | DHCP deployment and reservation | 10 screenshots |
| [08](08-File-Server) | File server, AGDLP, and access control | 28 screenshots |
| [09](09-FSRM) | Quotas, file screening, and storage reports | 11 screenshots |
| [10](10-Shadow-Copies) | Shadow Copies and file recovery | 6 screenshots |
| [11](11-Windows-Server-Backup) | Scheduled backup and file recovery | 8 screenshots |

The repository currently contains **151 screenshots** across eleven documented phases. The reviewed Phase 01–05 documents explicitly distinguish implementation claims from the evidence actually captured.

## Architecture and scope

This is a consolidated lab, not a production reference architecture. One server hosts several infrastructure roles, and the VMware host-only network intentionally has no default gateway. That design keeps the exercise self-contained but does not provide redundancy, high availability, internet name resolution, or production isolation between roles.

Additional boundaries:

- `virexon.local` is used only inside this closed lab.
- Example identities and organization data are fictional.
- The documentation does not intentionally publish passwords or reusable credentials.
- Shadow Copies support convenient file recovery but are not a substitute for an independent backup.
- The dedicated Windows Server Backup disk is suitable for lab validation but does not provide production-grade off-host or off-site protection.

## Documentation integrity note

The Phase 01 system screenshot identifies the server as **Windows Server 2025 Standard Evaluation**. Some legacy wording in later phase documents still refers to Windows Server 2022; the screenshot is treated as the source of truth, and that wording is scheduled for reconciliation in the next reviewed documentation batch.

## Status

Phases 01–11 are implemented and documented. Documentation is being reviewed in controlled batches so technical corrections can be validated without changing the completed lab design or its evidence set.
