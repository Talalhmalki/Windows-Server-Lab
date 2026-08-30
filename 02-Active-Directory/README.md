# 02 - Active Directory Domain Services

## Purpose

This phase establishes centralized identity services for the lab by installing Active Directory Domain Services (AD DS), creating the `virexon.local` forest, and promoting `PC26` to a domain controller.

## Implemented state

| Component | Implementation |
| --- | --- |
| Forest and domain | `virexon.local` |
| NetBIOS domain name | `VIREXON` |
| Domain controller | `PC26` |
| Deployment model | New, single-domain forest for an isolated lab |
| Administration console | Active Directory Users and Computers (ADUC) |

## Deployment workflow

1. Started the **Add Roles and Features Wizard** from Server Manager.
2. Selected a role-based or feature-based installation.
3. Selected the local server as the deployment target.
4. Selected the **Active Directory Domain Services** role.
5. Completed domain-controller promotion and verified the resulting directory in ADUC.

## Evidence boundary

The available screenshots capture role selection and the final ADUC state. They do not capture every page of the domain-controller promotion wizard. The completed domain is supported by the ADUC result below and by downstream evidence, including the Windows 11 domain join in [Phase 04](../04-Users-and-Groups) and domain DNS validation in [Phase 06](../06-DNS).

## Validation evidence

| Stage | Evidence | Validation |
| --- | --- | --- |
| Wizard launch | [01 - Add Roles and Features Wizard](Screenshots/01-Add-Roles-and-Features-Wizard.png) | Role deployment workflow opened. |
| Installation type | [02 - Installation Type](Screenshots/02-Installation-Type.png) | Role-based installation selected. |
| Target server | [03 - Server Selection](Screenshots/03-Server-Selection.png) | Local server selected for deployment. |
| AD DS role | [04 - Server Roles](Screenshots/04-Server-Roles-ADDS.png) | AD DS selected in the role list. |
| Post-deployment state | [05 - Active Directory Users and Computers](Screenshots/05-Active-Directory-Users-and-Computers.png) | `virexon.local` and its default directory containers are visible in ADUC. |

## Lab constraint

`PC26` is the only domain controller because this is a consolidated learning environment. A production design would normally use multiple domain controllers, tested restore procedures, monitored replication, and role placement appropriate to the organization's availability and security requirements.

## Outcome

The `virexon.local` directory is operational and ready for the OU structure, identities, computers, and policies implemented in the following phases.
