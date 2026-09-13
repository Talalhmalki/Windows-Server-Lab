# Windows Server Infrastructure Lab - VIREXON

A screenshot-backed Windows Server lab that demonstrates the progressive design, implementation, and validation of core enterprise infrastructure services for the fictitious **VIREXON** organization. The environment begins with a single Domain Controller and develops into a two-DC Active Directory lab with centralized identity, policy, networking, file services, recovery, hardening, firewall management, and auditing.

## Current environment

| Component | Verified implementation |
| --- | --- |
| Organization | VIREXON (fictitious) |
| Active Directory domain | `virexon.local` |
| NetBIOS name | `VIREXON` |
| Active Directory site | `Riyadh-HQ` |
| Network | VMware VMnet1 host-only network, `192.168.1.0/24` |
| Primary Domain Controller | `PC26.virexon.local` — `192.168.1.2` |
| Additional Domain Controller | `PC27.virexon.local` — `192.168.1.3` |
| Administrative client | `PC-IT-01` — Windows 11 Pro, reserved address `192.168.1.50` |
| Department share | `\\PC26\Departments` backed by `F:\CompanyData\Departments` |
| Virtualization | VMware Workstation Pro |

`PC26` provides AD DS, DNS, DHCP, departmental file services, and holds all five FSMO roles in the final documented state. `PC27` is an additional writable Domain Controller, DNS server, and Global Catalog. Both Domain Controllers are members of `Riyadh-HQ` and participate in Active Directory replication.

The Phase 01 evidence identifies `PC26` as Windows Server 2025 Standard Evaluation. Later server captures use the Windows Server 2025 platform as well. Each phase describes the environment as it existed at that point in the implementation.

## What the project demonstrates

- Active Directory Domain Services deployment and two-Domain-Controller validation.
- Department-based OU, user, computer, and security-group administration.
- Group Policy design, scoping, positive testing, and negative testing.
- DNS forward and reverse resolution, AD service discovery, and DNS-integrated replication.
- DHCP scope, exclusion, reservation, and client lease validation.
- Department file services using AGDLP, SMB, share permissions, and NTFS permissions.
- File Server Resource Manager quotas, file screening, and storage reporting.
- Shadow Copies and Windows Server Backup recovery workflows.
- Delegated Active Directory administration and privileged-account separation.
- Active Directory Sites and Services, client site detection, and DC authentication failover.
- Planned FSMO transfer and restoration between healthy Domain Controllers.
- System State backup and non-authoritative Domain Controller recovery.
- Staged Domain Controller hardening with post-change service validation.
- A controlled Windows Defender Firewall pilot on `PC27`.
- Advanced Audit Policy deployment to both Domain Controllers and event validation.

## Completed phases

| Phase | Area | Evidence |
| --- | --- | ---: |
| [01](01-Server-Preparation) | Server Baseline and Environment Verification | 6 screenshots |
| [02](02-Active-Directory) | Active Directory Domain Services | 5 screenshots |
| [03](03-Organizational-Units) | Organizational Unit Design | 4 screenshots |
| [04](04-Users-and-Groups) | Users, Groups, and Domain Client | 11 screenshots |
| [05](05-Group-Policy) | Group Policy | 46 screenshots |
| [06](06-DNS) | Domain Name System (DNS) | 16 screenshots |
| [07](07-DHCP) | Dynamic Host Configuration Protocol (DHCP) | 10 screenshots |
| [08](08-File-Server) | File Server and Access Control | 28 screenshots |
| [09](09-FSRM) | File Server Resource Manager (FSRM) | 11 screenshots |
| [10](10-Shadow-Copies) | Shadow Copies | 6 screenshots |
| [11](11-Windows-Server-Backup) | Windows Server Backup | 8 screenshots |
| [12](12-Advanced-Active-Directory) | Advanced Active Directory | 22 screenshots |
| [13](13-Active-Directory-Sites-and-Services) | Active Directory Sites and Services | 9 screenshots |
| [14](14-FSMO-Roles) | FSMO Roles | 6 screenshots |
| [15](15-Active-Directory-Backup-and-Recovery) | Active Directory Backup and Recovery | 9 screenshots |
| [16](16-Security-and-Server-Hardening) | Security and Server Hardening | 16 screenshots |
| [17](17-Windows-Defender-Firewall) | Windows Defender Firewall | 9 screenshots |
| [18](18-Advanced-Auditing) | Advanced Auditing | 14 screenshots |

The repository contains **236 screenshots across 18 completed phases**. Each completed phase links every retained screenshot to a precise, evidence-bounded explanation.

## Architecture evolution

The implementation is cumulative:

1. Phases 01–12 establish and extend the original `PC26`-based environment.
2. Phase 13 introduces `PC27` and converts the directory layer to a two-DC design.
3. Phase 14 temporarily transfers all five FSMO roles to `PC27`, then restores the intended final placement on `PC26`.
4. Phase 15 validates System State recovery of `PC27` with `PC26` available as its replication partner.
5. Phase 16 applies staged hardening to both Domain Controllers.
6. Phase 17 remains intentionally limited to a firewall pilot on `PC27`.
7. Phase 18 deploys the selected Advanced Audit Policy settings to both Domain Controllers.

Statements in earlier phase documents describe the verified state at that stage. For example, Phase 06 validates the original single-DNS-server design; Phase 13 subsequently adds the second DNS-enabled Domain Controller.

## Scope and design boundaries

This repository documents a hands-on lab, not a production reference architecture.

- Both Domain Controllers run as virtual machines on the same VMware host; the lab therefore does not protect against physical-host failure.
- DHCP and departmental file services remain on `PC26` and are not demonstrated as highly available.
- The Phase 13 outage test validates domain authentication through `PC27`, not failover of DHCP or file services.
- The Phase 17 firewall GPO remains scoped to `PC27`; no full two-DC firewall rollout is claimed.
- Backup targets are separate virtual disks attached within the same lab infrastructure, not off-host or off-site repositories.
- `virexon.local` is used only inside the isolated lab.
- Identities and organization data are fictional.
- Screenshots are treated as evidence of the visible result; the documentation avoids claiming uncaptured actions or unsupported root causes.

## Documentation standard

The phase reports use a consistent evidence model:

- configuration claims are tied to the relevant screenshot;
- successful and denied actions are distinguished;
- temporary errors are recorded separately from final health results;
- a passing state is not described as broader availability than the test proves;
- sensitive values such as passwords are not intentionally published;
- planned work is not presented as completed work.

## Roadmap

Phase 19, **PowerShell Administration**, is the next planned implementation. Repository folders for later phases are placeholders and are not included in the completed-phase or screenshot totals.

## Status

Phases 01–18 are implemented, evidence-backed, and documentation-reviewed. Phase 19 has not yet been claimed as complete.
