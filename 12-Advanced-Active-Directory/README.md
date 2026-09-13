# 12 - Advanced Active Directory

## Purpose

This phase extends the `virexon.local` lab with delegated administration, separate privileged identities, a Fine-Grained Password Policy, restricted administrative sign-in, Organizational Unit protection, and deleted-object recovery. Each permission boundary is supported by a permitted or denied operation rather than by configuration alone.

## Verified environment

| Component | Configuration |
| --- | --- |
| Domain | `virexon.local` |
| Domain Controller | `PC26.virexon.local` — `192.168.1.2` |
| Administrative workstation | `PC-IT-01` — `192.168.1.50` |
| Server platform | Windows Server 2025 Standard Evaluation |
| Client platform | Windows 11 Pro |
| Remote administration | RSAT, Active Directory Users and Computers, and Active Directory Administrative Center |
| Evidence | 22 screenshots |

## Administrative model

Administrative accounts were separated from standard user accounts and placed in `Administrative-Accounts`. Permissions were assigned through role groups instead of directly to individual users.

| Role | Test account | Role group | Intended scope |
| --- | --- | --- | --- |
| Help Desk | `hd-ahmed.ali` | `GG-HelpDesk-Operators` | Standard-user password resets and account-unlock attributes |
| AD Operator | `adm-mohammed.saleh` | `GG-AD-Operators` | Standard-user administration and departmental group membership |
| Privileged AD administrator | `adm-sami.ahmed` | `GG-AD-Admins` | Privileged directory administration |
| Standard daily user | `s.ahmed` | Standard user groups | Routine work without administrative authority |

The separation between `Administrative-Accounts` and the delegated `Users` hierarchy prevents standard delegation from inheriting onto privileged identities.

## RSAT administration

The AD DS RSAT components were installed on `PC-IT-01`. A remote session running as `VIREXON\Administrator` opened the domain directory from that workstation, confirming that administration did not require an interactive session on the Domain Controller.

## Delegated administration

### Help Desk

`GG-HelpDesk-Operators` received password-reset and account-unlock-related permissions for descendant user objects beneath `VIREXON\Users`. The scope excludes `Administrative-Accounts`.

The positive test reset the password of the standard account for Tariq Alotaibi. The negative test attempted to change the password of the privileged Mohammed Saleh administrative account and returned `Access is denied`.

### AD Operator

`GG-AD-Operators` received two separate delegation scopes:

- create, delete, and manage standard users beneath `VIREXON\Users`; and
- modify membership of groups beneath `VIREXON\Groups\Department-Groups`.

The permitted-action evidence shows `AD Operator Test User` added to `GG-IT-Users`. The privilege-boundary test shows that the Add and Remove controls for `GG-AD-Admins` were unavailable to the delegated operator.

### Privileged administrator and daily-account separation

`GG-AD-Admins` was added to the built-in `Domain Admins` group for this lab scenario, and `adm-sami.ahmed` received privileged authority through that role group. The dedicated administrative account created a privileged test user successfully.

The related daily account, `s.ahmed`, was denied when it attempted to change the privileged test user's password. This demonstrates that privileged authority was not assigned to the routine identity.

## Fine-Grained Password Policy

The Password Settings Object `PSO-Privileged-Admins` was applied directly to `GG-AD-Admins`.

| Setting | Verified value |
| --- | --- |
| Precedence | `1` |
| Minimum password length | 14 characters |
| Password history | 24 passwords |
| Complexity | Enabled |
| Reversible encryption | Disabled |
| Minimum password age | 1 day |
| Maximum password age | 30 days |
| Failed sign-in threshold | 5 attempts |
| Reset failed-attempt counter | 15 minutes |
| Lockout duration | 15 minutes |
| Accidental deletion protection | Enabled |

Active Directory Administrative Center displayed `PSO-Privileged-Admins` as the resultant password policy for `adm-sami.ahmed`. A subsequent password-change attempt was rejected by the effective password policy.

The rejection dialog does not expose the candidate password itself. Therefore, the evidence supports policy enforcement when read with the saved 14-character configuration and resultant-policy capture; it does not independently prove the exact length or composition of the rejected value.

## Administrative workstation restriction

The `Log On To` restriction for `adm-sami.ahmed` was limited to `PC-IT-01`. An interactive sign-in on another computer was denied with the message that the account was configured to prevent use of that PC.

This is an account-level workstation restriction within the lab. It is not presented as a complete privileged-access workstation architecture.

## Directory protection and recovery

### Accidental deletion protection

The `Administrative-Accounts` OU was configured with **Protect object from accidental deletion**. A subsequent deletion attempt was denied and explicitly referenced the protection setting.

### Active Directory Recycle Bin

A temporary account named `test` was created in the IT OU, added to `GG-IT-Users`, deleted, located in **Deleted Objects**, and restored. The restored object returned to its original OU with both `Domain Users` and `GG-IT-Users` memberships visible.

Deletion protection and Recycle Bin address different risks: the first blocks an unintended deletion, while the second supports recovery after deletion.

## Access-control validation

| Operation | Help Desk | AD Operator | Privileged administrator | Daily user |
| --- | :---: | :---: | :---: | :---: |
| Reset a standard-user password | Verified | Delegated | Allowed by role | Not delegated |
| Manage standard users | Not delegated | Delegated | Allowed by role | Not delegated |
| Modify departmental group membership | Not delegated | Verified | Allowed by role | Not delegated |
| Modify the privileged role group | Outside delegated scope | Denied by tested boundary | Allowed by role | Not delegated |
| Manage privileged accounts | Denied by tested boundary | Outside delegated scope | Verified | Denied by tested action |
| Restore deleted directory objects | Not delegated | Not delegated | Verified | Not delegated |

`Configured` and `verified` are intentionally distinguished: an assigned permission is not described as functionally tested unless a corresponding action appears in the retained evidence.

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [Administrative Role Structure](Screenshots/01-Administrative-Role-Structure-Verification.png) | Separate administrative identities are present in `Administrative-Accounts`. |
| 02 | [RSAT AD DS Tools Installation](Screenshots/02-RSAT-AD-DS-Tools-Installation-Verification.png) | AD DS RSAT components are installed on the workstation. |
| 03 | [RSAT Remote Administration](Screenshots/03-RSAT-Remote-AD-Administration-Verification.png) | `PC-IT-01` can open the domain directory remotely. |
| 04 | [Help Desk Delegation](Screenshots/04-HelpDesk-Role-and-Delegation-Configuration.png) | `GG-HelpDesk-Operators` has the documented standard-user delegation. |
| 05 | [Help Desk Allowed Action](Screenshots/05-HelpDesk-Allowed-Action-Verification.png) | The Help Desk account can reset a standard user's password. |
| 06 | [Help Desk Boundary](Screenshots/06-HelpDesk-Administrative-Account-Access-Denied.png) | The same account is denied against a privileged administrative identity. |
| 07 | [AD Operator User Delegation](Screenshots/07-AD-Operator-User-Delegation-Configuration.png) | Standard-user lifecycle permissions are assigned to `GG-AD-Operators`. |
| 08 | [AD Operator Group Delegation](Screenshots/08-AD-Operator-Group-Membership-Delegation.png) | Departmental group-membership modification is delegated separately. |
| 09 | [AD Operator Allowed Action](Screenshots/09-AD-Operator-Allowed-Actions-Verification.png) | The operator can add the test user to `GG-IT-Users`. |
| 10 | [AD Operator Escalation Boundary](Screenshots/10-AD-Operator-Privilege-Escalation-Denied.png) | Membership controls for `GG-AD-Admins` are unavailable to the operator. |
| 11 | [Privileged Role Assignment](Screenshots/11-Privileged-AD-Admin-Role-Assignment.png) | `GG-AD-Admins` is a member of `Domain Admins`. |
| 12 | [Privileged Action](Screenshots/12-AD-Admin-Privileged-Action-Verification.png) | `adm-sami.ahmed` can create a privileged test user. |
| 13 | [Daily-Account Boundary](Screenshots/13-Daily-Account-Administrative-Action-Denied.png) | `s.ahmed` is denied a privileged password-management action. |
| 14 | [Privileged PSO Configuration](Screenshots/14-Privileged-PSO-Configuration.png) | The PSO scope and password/lockout values are saved. |
| 15 | [Resultant PSO](Screenshots/15-Privileged-PSO-Resultant-Policy.png) | The privileged administrator resolves to `PSO-Privileged-Admins`. |
| 16 | [PSO Enforcement](Screenshots/16-Privileged-PSO-Enforcement-Verification.png) | A noncompliant password-change attempt is rejected by policy. |
| 17 | [Workstation Restriction](Screenshots/17-Administrative-Workstation-Restriction-Configuration.png) | The privileged account is restricted to `PC-IT-01`. |
| 18 | [Unauthorized Workstation Test](Screenshots/18-Unauthorized-Workstation-Logon-Denied.png) | Interactive sign-in from another computer is denied. |
| 19 | [OU Protection Configuration](Screenshots/19-Administrative-OU-Protection-Configuration.png) | Accidental deletion protection is enabled on `Administrative-Accounts`. |
| 20 | [OU Deletion Protection](Screenshots/20-Accidental-Deletion-Protection-Verification.png) | The protected OU deletion attempt is denied. |
| 21 | [Deleted Object](Screenshots/21-AD-Recycle-Bin-Deleted-Object-Verification.png) | The deleted test account is recoverable from Deleted Objects. |
| 22 | [Object Restoration](Screenshots/22-AD-Recycle-Bin-Object-Restoration-Verification.png) | The account returns to its original OU with retained memberships. |

## Operational considerations

- Permanent membership in `Domain Admins` should be minimized in production and monitored closely.
- Delegation should be reviewed periodically and paired with auditing.
- Dedicated administrative accounts should not be used for routine browsing, email, or productivity work.
- Workstation restrictions complement, but do not replace, a hardened privileged-access workstation design.
- Active Directory Recycle Bin does not replace System State backup or forest-recovery planning.
- Test accounts and temporary privileged objects should be removed after validation.

## Outcome

The lab now separates daily and administrative identities, assigns permissions through scoped role groups, enforces a targeted privileged password policy, restricts privileged sign-in, protects a critical OU, and restores deleted directory objects. Both permitted and denied actions confirm the intended authorization boundaries.

**12 - Advanced Active Directory — Completed ✅**
