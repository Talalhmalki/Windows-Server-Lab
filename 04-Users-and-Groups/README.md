# 04 - Users, Groups, and Domain Client

## Purpose

This phase implements a representative identity and client-management workflow in Active Directory: create IT user accounts, create department-based Global security groups, assign the IT users to their role group, join a Windows 11 client to the domain, and place the computer object in the correct OU.

## Implemented objects

| Object type | Verified implementation |
| --- | --- |
| User accounts | Six representative IT accounts under `VIREXON\Users\IT`, with role descriptions and example contact attributes. |
| Global security groups | `GG-IT-Users`, `GG-HR-Users`, `GG-Finance-Users`, `GG-Marketing-Users`, `GG-Sales-Users`, and `GG-Management-Users`. |
| Group membership | The six IT accounts were added to `GG-IT-Users`. The screenshots verify creation, but not populated membership, for the other department groups. |
| Client computer | `PC-IT-01`, running Windows 11 Pro 25H2, joined to `virexon.local`. |
| Computer placement | `PC-IT-01` moved to `VIREXON\Computers\IT`. |

## Group design and AGDLP

This phase implements the first two elements of the AGDLP model:

1. **A — Accounts:** IT user accounts.
2. **G — Global groups:** `GG-IT-Users` and the other department identity groups.

It does not, by itself, complete AGDLP. Domain Local resource groups and permissions—the **DL** and **P** elements—are implemented and validated in [Phase 08](../08-File-Server).

## Evidence notes

### Historical account-creation captures

[Screenshot 01](Screenshots/01-New-User-Wizard.png) and [Screenshot 02](Screenshots/02-User-Password-Configuration.png) show the New Object wizard opened while `VIREXON\Computers\IT` was selected. They are retained as historical workflow evidence, but they are not used to prove final user placement. [Screenshot 04](Screenshots/04-Users-Created-and-Organized.png) is the final-state evidence and shows the accounts under `VIREXON\Users\IT`.

### Password-expiration exception

The captured account-creation example has **Password never expires** selected. That setting is documented here as a lab exception, not a production recommendation. It prevents the account from being governed by maximum-password-age expiration and should be explicitly reviewed or cleared when evaluating the password policy in [Phase 05](../05-Group-Policy). Microsoft documents the setting as the `DONT_EXPIRE_PASSWD` account-control flag: [UserAccountControl flags](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties).

## Validation evidence

| Area | Evidence | What it verifies |
| --- | --- | --- |
| Account workflow | [01 - New User Wizard](Screenshots/01-New-User-Wizard.png), [02 - Password Configuration](Screenshots/02-User-Password-Configuration.png) | Historical creation workflow and captured account options; not final OU placement. |
| Account attributes | [03 - User Properties](Screenshots/03-User-Properties-General.png) | Representative user description and contact attributes. |
| Final user placement | [04 - Users Created and Organized](Screenshots/04-Users-Created-and-Organized.png) | Six accounts visible under `VIREXON\Users\IT`. |
| Group creation | [05 - New Security Group Wizard](Screenshots/05-New-Security-Group-Wizard.png), [06 - Security Groups Created](Screenshots/06-Security-Groups-Created.png) | Global security-group type and six department group objects. |
| Membership change | [07 - Before Membership](Screenshots/07-Group-Membership-Before-Adding-Users.png), [08 - Membership Configured](Screenshots/08-Group-Membership-Configured.png) | `GG-IT-Users` changed from empty to six IT members. |
| Client identity | [09 - Computer Information](Screenshots/09-Computer-Information.png) | `PC-IT-01`, Windows 11 Pro 25H2, and the `virexon.local` FQDN. |
| AD computer placement | [10 - Computer Moved to IT OU](Screenshots/10-Computer-Moved-To-IT-OU.png) | Computer object located under `VIREXON\Computers\IT`. |
| Domain membership | [11 - Computer Domain Membership](Screenshots/11-Computer-Domain-Membership.png) | Client reports membership in `virexon.local`. |

## Outcome

The lab now has verified IT identities, reusable department Global groups, and a managed Windows client in the intended computer OU. This provides the identity and targeting foundation used by Group Policy in [Phase 05](../05-Group-Policy) and resource authorization in [Phase 08](../08-File-Server).
