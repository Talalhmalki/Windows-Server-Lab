# Phase 18 — Advanced Auditing

## Overview

This phase implements a controlled Windows Server auditing configuration for the VIREXON Active Directory environment.

The objective was not to build a complete SOC, SIEM, or enterprise-wide security monitoring platform. Instead, the phase focused on practical auditing capabilities that a Windows System Administrator should understand:

- Centrally configuring selected Advanced Audit Policy subcategories through Group Policy.
- Verifying the effective audit policy on the target Domain Controllers.
- Generating controlled Active Directory account-management activity.
- Validating the resulting Security events in Event Viewer.
- Using a pilot-first deployment model before expanding the configuration.
- Confirming that Active Directory replication remained healthy after deployment.

The implementation followed the validation flow:

**Group Policy → Advanced Audit Policy → Effective Audit Policy → Security Event → Event Viewer Verification**

---

## Environment

| Component | Configuration |
|---|---|
| Domain | `virexon.local` |
| NetBIOS Name | `VIREXON` |
| Active Directory Site | `Riyadh-HQ` |
| Network | `192.168.1.0/24` |
| Primary Domain Controller | `PC26.virexon.local` |
| PC26 IP Address | `192.168.1.2` |
| Additional Domain Controller | `PC27.virexon.local` |
| PC27 IP Address | `192.168.1.3` |
| Administrative Workstation | `PC-IT-01` |
| PC-IT-01 Reserved IP | `192.168.1.50` |
| Virtualization Platform | VMware Workstation Pro |

---

## Business Requirement

VIREXON required important security-related activity on its Domain Controllers to be recorded consistently and managed centrally.

The required auditing visibility included:

- Successful and failed logon activity.
- User account management activity.
- Active Directory user-account creation.
- Active Directory user-account deletion.

The implementation also needed to avoid unnecessary high-volume audit categories and preserve Domain Controller and Active Directory replication health.

---

## Technical Design

A dedicated Group Policy Object was created:

`GPO - DC Advanced Auditing`

The GPO was linked to:

`virexon.local/Domain Controllers`

The deployment used a controlled pilot approach.

### Pilot Scope

Initial Security Filtering:

- `PC27$`

PC27 was used as the pilot Domain Controller.

PC26 remained outside the auditing GPO until the PC27 validation was completed successfully.

### Final Scope

After successful PC27 validation, the Security Filtering was expanded to:

- `PC26$`
- `PC27$`

The following existing policies were not modified during this phase:

- Default Domain Policy
- Default Domain Controllers Policy
- `GPO - DC Disable Print Spooler`
- `GPO - DC Windows Defender Firewall`

---

## Advanced Audit Policy Configuration

Only two Advanced Audit Policy subcategories were configured.

### Logon / Logoff

**Audit Logon**

Configured as:

- Success: Enabled
- Failure: Enabled

Purpose:

To centrally enforce auditing for successful and failed logon activity handled by the server.

Typical related Security events include:

- Event ID `4624` — successful logon
- Event ID `4625` — failed logon

The effective Audit Logon policy was verified on both Domain Controllers.

However, this phase did **not** deliberately perform a dedicated functional test for both Event ID 4624 and Event ID 4625.

Therefore, the documentation does not claim that both successful and failed logon events were functionally tested.

---

### Account Management

**Audit User Account Management**

Configured as:

- Success: Enabled
- Failure: Enabled

Purpose:

To record important user-account management activity such as:

- User creation
- User deletion
- Account changes
- Password-related management actions
- Account enable/disable activity

For this phase, user creation and deletion were functionally tested.

---

## Advanced Audit Policy Override

The following Security Option was enabled:

**Audit: Force audit policy subcategory settings (Windows Vista or later) to override audit policy category settings**

Value:

`Enabled`

This ensures that the Advanced Audit Policy subcategory configuration is used instead of being unintentionally overridden by legacy/basic audit-category settings.

---

## Pre-Deployment Baseline

Before the dedicated auditing GPO was applied to PC27, the effective audit policy was reviewed using `auditpol`.

The baseline showed:

| Subcategory | Baseline State |
|---|---|
| Audit Logon | Success and Failure |
| Audit User Account Management | Success |

This means that some auditing already existed before Phase 18.

The baseline was documented only as the pre-existing effective state.

Phase 18 does **not** claim that every auditing setting was created from an unconfigured state.

The dedicated GPO was used to centrally enforce the required design and ensure that User Account Management auditing included both Success and Failure.

---

## PC27 Pilot Deployment

The auditing GPO was initially restricted to:

`PC27$`

After Group Policy refresh, `gpresult` confirmed that:

`GPO - DC Advanced Auditing`

appeared under:

`Applied Group Policy Objects`

on PC27.

The effective audit configuration was then verified with `auditpol`.

PC27 showed:

- `Logon = Success and Failure`
- `User Account Management = Success and Failure`

This confirmed that the intended Advanced Audit Policy configuration was effective on the pilot Domain Controller.

---

## Controlled User Account Audit Test

A temporary non-privileged Active Directory account was used for functional validation.

### Temporary Account

Display Name:

`Audit Test User`

Logon Name:

`audit.test`

The account was created only for auditing validation.

It was not granted administrative privileges and was not added to privileged groups.

The account operation was performed through Active Directory Users and Computers while targeting PC27 as the Domain Controller handling the change.

---

## User Creation Audit Validation

After creating the temporary user, the Security log on PC27 was reviewed.

The following event was successfully located and verified:

**Event ID 4720 — A user account was created**

The event confirmed:

- Computer: `PC27.virexon.local`
- Target Account: `audit.test`
- Security ID: `VIREXON\audit.test`
- Display Name: `Audit Test User`
- Task Category: User Account Management
- Audit Result: Success

This provided functional evidence that the User Account Management audit configuration was recording account-creation activity.

---

## User Deletion Audit Validation

The temporary account was then deleted.

The Security log on PC27 was reviewed again.

The following event was successfully located and verified:

**Event ID 4726 — A user account was deleted**

The event confirmed:

- Computer: `PC27.virexon.local`
- Target Account: `audit.test`
- Security ID: `VIREXON\audit.test`
- Task Category: User Account Management
- Audit Result: Success

The temporary test account was therefore removed after completing the controlled validation.

No temporary administrative or privileged account was created during this phase.

---

## PC27 Pilot Replication Validation

Active Directory replication was checked before the auditing changes and again after the PC27 pilot.

The initial replication baseline showed:

- PC26 Source: 0 failures
- PC27 Source: 0 failures
- PC26 Destination: 0 failures
- PC27 Destination: 0 failures
- Failure percentage: 0%

During the post-pilot validation, a temporary replication issue was observed.

`repadmin /replsummary` reported:

`1722 — The RPC server is unavailable`

The failure affected replication from PC26 toward PC27 for part of the replication topology.

Additional inspection with `repadmin /showrepl` showed that some naming contexts were successfully replicating while the Configuration and Schema naming contexts had recorded RPC failures.

The issue was treated as a technical validation blocker and no further auditing deployment was performed until replication health returned to normal.

A later replication validation returned:

- PC26 Source: 0 failures
- PC27 Source: 0 failures
- PC26 Destination: 0 failures
- PC27 Destination: 0 failures
- Failure percentage: 0%

No root cause for the temporary RPC replication failure was proven.

Therefore, this phase does **not** claim that the auditing configuration caused the replication issue.

The condition was observed, validated, and confirmed healthy before continuing the deployment.

---

## Final Deployment to PC26

After the PC27 pilot was successfully validated and replication returned to a healthy state, PC26 was added to the GPO Security Filtering.

Final Security Filtering:

- `PC26$`
- `PC27$`

After the policy refresh on PC26, `gpresult` confirmed that:

`GPO - DC Advanced Auditing`

was listed under:

`Applied Group Policy Objects`

The effective audit policy was then verified with `auditpol`.

PC26 showed:

- `Logon = Success and Failure`
- `User Account Management = Success and Failure`

This confirmed that the final auditing configuration was effective on both Domain Controllers.

---

## Firewall Policy Separation

During validation on PC26, `gpresult` also showed:

`GPO - DC Windows Defender Firewall`

under the policies that were not applied because of Security Filtering.

This is expected.

Phase 17 intentionally retained the Windows Defender Firewall GPO as a PC27-only pilot deployment.

The Firewall GPO was not expanded or modified during Phase 18.

---

## Final Active Directory Health Validation

After the auditing configuration was deployed to both Domain Controllers, a final Active Directory replication validation was performed using:

`repadmin /replsummary`

The final result showed:

### Source DSA

- PC26: `0 / 5` failures
- PC27: `0 / 5` failures

### Destination DSA

- PC26: `0 / 5` failures
- PC27: `0 / 5` failures

Final failure percentage:

`0%`

No replication errors were present in the final validation.

This confirmed that both Domain Controllers remained healthy after the Phase 18 deployment.

---

## GPO Backup Status

A backup of:

`GPO - DC Advanced Auditing`

was included in the original deployment plan before expanding the policy to PC26.

The backup was intentionally **not performed**.

Therefore, this README does not claim that a GPO backup exists.

The dedicated GPO design still provides configuration isolation from previous project phases, and Security Filtering can be used to remove an individual Domain Controller from the auditing deployment if rollback is required.

---

## What Was Functionally Tested

The following items were functionally validated:

| Test | Result |
|---|---|
| Auditing GPO applied to PC27 | Passed |
| Effective Audit Logon policy on PC27 | Passed |
| Effective User Account Management policy on PC27 | Passed |
| Event ID 4720 generated for `audit.test` | Passed |
| Event ID 4726 generated for `audit.test` | Passed |
| Temporary user deleted | Passed |
| PC27 pilot replication health | Passed after temporary RPC failure cleared |
| Auditing GPO applied to PC26 | Passed |
| Effective Audit Logon policy on PC26 | Passed |
| Effective User Account Management policy on PC26 | Passed |
| Final AD replication health | Passed |
| Final replication failure percentage | 0% |

---

## Configured but Not Functionally Tested

The following policy was configured and verified as effective:

`Audit Logon = Success and Failure`

However, this phase did not intentionally generate dedicated test scenarios for both:

- Event ID 4624
- Event ID 4625

Therefore:

**Audit Logon was configured and verified as effective, but successful and failed logon events were not both deliberately functionally tested as part of this phase.**

---

## Out of Scope

The following auditing and security technologies were intentionally excluded from Phase 18:

- File and folder Object Access Auditing
- NTFS SACL configuration
- File Server access auditing
- Audit Process Creation
- Command-line process auditing
- PowerShell Script Block Logging
- PowerShell Transcription
- Registry auditing
- Directory Service Changes / Event ID 5136
- Deep Kerberos auditing
- Windows Event Forwarding
- SIEM integration
- Microsoft Sentinel
- Microsoft Defender for Identity
- Sysmon
- Centralized audit-log forwarding
- Full Microsoft enterprise auditing baseline
- Account lockout testing
- Repeated incorrect-password testing

These items were excluded intentionally to keep Phase 18 focused on practical Windows Server administration rather than SOC or SIEM engineering.

---

## Evidence

The implementation was documented using the following 14 screenshots.

| # | Screenshot | Evidence |
|---|---|---|
| 01 | `01-Pre-Auditing-AD-Replication-Health.png` | Proves AD replication was healthy before Phase 18 changes. |
| 02 | `02-Pre-Auditing-PC27-Audit-Policy-Baseline.png` | Documents the effective PC27 auditing baseline before the dedicated GPO was applied. |
| 03 | `03-DC-Auditing-GPO-Pilot-Scope.png` | Shows the dedicated auditing GPO linked to the Domain Controllers OU with PC27-only Security Filtering. |
| 04 | `04-DC-Advanced-Audit-Policy-Configuration.png` | Shows Audit Logon and Audit User Account Management configured for Success and Failure. |
| 05 | `05-DC-Audit-Subcategory-Override-Configuration.png` | Shows the Advanced Audit Policy subcategory override Security Option enabled. |
| 06 | `06-PC27-Auditing-Policy-Result.png` | Confirms the auditing GPO was applied to PC27. |
| 07 | `07-PC27-Effective-Audit-Policy.png` | Confirms the effective Advanced Audit Policy on PC27. |
| 08 | `08-PC27-User-Creation-Audit-Event.png` | Shows Event ID 4720 for creation of `audit.test`. |
| 09 | `09-PC27-User-Deletion-Audit-Event.png` | Shows Event ID 4726 for deletion of `audit.test`. |
| 10 | `10-PC27-Pilot-AD-Replication-Health.png` | Confirms replication returned to 0 failures after the PC27 pilot. |
| 11 | `11-DC-Auditing-GPO-Final-Scope.png` | Shows final Security Filtering containing both PC26 and PC27. |
| 12 | `12-PC26-Auditing-Policy-Result.png` | Confirms the auditing GPO was applied to PC26. |
| 13 | `13-PC26-Effective-Audit-Policy.png` | Confirms the effective Advanced Audit Policy on PC26. |
| 14 | `14-Final-Auditing-AD-Replication-Health.png` | Final proof that replication remained healthy with 0 failures after full deployment. |

---

## Final Configuration

Final GPO:

`GPO - DC Advanced Auditing`

Linked to:

`virexon.local/Domain Controllers`

Final Security Filtering:

- `PC26$`
- `PC27$`

Final configured Advanced Audit Policy:

- Audit Logon: Success and Failure
- Audit User Account Management: Success and Failure

Security Option:

- Force audit policy subcategory settings to override audit policy category settings: Enabled

Functionally verified Security events:

- Event ID `4720` — User account created
- Event ID `4726` — User account deleted

Temporary test account:

`audit.test`

Status:

Deleted after validation.

Final Active Directory replication status:

`0 failures`

---

## Result

Phase 18 successfully demonstrated the complete Windows Server auditing workflow:

**Dedicated GPO → Advanced Audit Policy → Effective Policy → Controlled AD Activity → Security Event → Event Viewer Verification**

The auditing configuration was first validated on PC27 and then expanded to PC26 only after the pilot succeeded.

Selected Advanced Audit Policy subcategories were centrally configured and verified as effective on both Domain Controllers.

Controlled Active Directory user-account creation and deletion activity was successfully captured through Security Event IDs 4720 and 4726.

The temporary audit account was removed after testing.

The phase intentionally avoided unnecessary SOC/SIEM-level auditing features and remained focused on practical Windows Server System Administration.

Final Active Directory replication validation completed successfully with:

**0 replication failures.**

---

## Phase Status

**Phase 18 — Auditing: COMPLETED**

Next Phase:

**19 — PowerShell Administration**
