# 03 - Organizational Unit Design

## Purpose

This phase creates an administrative structure for users, computers, servers, groups, and service accounts in `virexon.local`. The hierarchy is designed to make object placement and Group Policy scope explicit without adding unnecessary OU depth.

## Implemented hierarchy

The custom hierarchy begins with the `VIREXON` root OU beneath the domain:

| Parent path | Direct child OUs |
| --- | --- |
| `virexon.local` | `VIREXON` |
| `VIREXON` | `Users`, `Computers`, `Servers`, `Groups`, `Service Accounts` |
| `VIREXON\Users` | `IT`, `HR`, `Finance`, `Sales`, `Management`, `Marketing` |
| `VIREXON\Computers` | `IT`, `HR`, `Finance`, `Sales`, `Management`, `Marketing` |

## OU responsibilities

| OU | Administrative purpose |
| --- | --- |
| `Users` | Contains departmental user OUs and provides a clear scope for user-side policies. |
| `Computers` | Contains departmental workstation OUs and separates computer-side policy targeting from user policy. |
| `Servers` | Keeps member-server objects outside workstation policy scope. |
| `Groups` | Centralizes security-group administration; permissions are assigned through groups in later phases. |
| `Service Accounts` | Separates non-human identities for distinct administration and lifecycle controls. |

## Design decisions

- The `VIREXON` root OU separates lab-managed objects from the domain's built-in containers.
- Matching department names under `Users` and `Computers` make policy targets predictable while keeping user and computer settings independent.
- The hierarchy is based on administrative and Group Policy requirements, not on an assumption that an OU alone grants access to resources.
- Resource authorization remains group-based; the complete AGDLP permission flow is implemented in [Phase 08](../08-File-Server).
- New OUs should be added only when they support a real delegation, policy, or object-management requirement.

## Validation evidence

| Evidence | What it verifies |
| --- | --- |
| [01 - Root Organizational Unit](Screenshots/01-Root-Organizational-Unit.png) | Creation of the `VIREXON` root OU. |
| [02 - Top-Level Organizational Units](Screenshots/02-Top-Level-Organizational-Units.png) | `Users`, `Computers`, `Servers`, `Groups`, and `Service Accounts` under `VIREXON`. |
| [03 - Users Department Structure](Screenshots/03-Users-Department-Structure.png) | Six departmental OUs under `VIREXON\Users`. |
| [04 - Computers Department Structure](Screenshots/04-Computers-Department-Structure.png) | Six departmental OUs under `VIREXON\Computers`. |

## Reference

The design follows Microsoft's principle of using OUs to organize objects for administration, delegation, and Group Policy application: [Reviewing OU design concepts](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/reviewing-ou-design-concepts).

## Outcome

The verified OU structure provides consistent placement targets for the identities and client computer created in [Phase 04](../04-Users-and-Groups) and the GPO links documented in [Phase 05](../05-Group-Policy).
