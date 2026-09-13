# 18 - Advanced Auditing

## Purpose

This phase deploys selected Advanced Audit Policy settings to the VIREXON Domain Controllers through Group Policy. The configuration is piloted on `PC27`, validated with controlled user-account events, and then expanded to `PC26` after replication returns a healthy result.

The scope is Windows Server administration and local Security-log validation. It is not presented as a SIEM, SOC, or complete enterprise audit baseline.

## Verified environment

| Component | Configuration |
| --- | --- |
| Domain | `virexon.local` |
| NetBIOS name | `VIREXON` |
| Active Directory site | `Riyadh-HQ` |
| Primary Domain Controller | `PC26.virexon.local` — `192.168.1.2` |
| Additional Domain Controller | `PC27.virexon.local` — `192.168.1.3` |
| GPO | `GPO - DC Advanced Auditing` |
| Final GPO targets | `PC26$` and `PC27$` |
| Evidence | 14 screenshots |

## Baseline and deployment sequence

The pre-change `repadmin /replsummary` result shows `0 / 5` source and destination failures for both DCs.

Before the new GPO, `auditpol` on `PC27` already reported:

| Subcategory | Baseline |
| --- | --- |
| Logon | Success and Failure |
| User Account Management | Success |

The dedicated GPO therefore standardizes the selected settings and adds failure auditing for User Account Management; it does not claim that all auditing was previously disabled.

Deployment followed this order:

1. Link the GPO to the `Domain Controllers` OU.
2. Restrict the pilot Security Filtering to `PC27$`.
3. Apply and validate the effective policy on `PC27`.
4. Generate and inspect controlled account-management events.
5. Revalidate replication.
6. Add `PC26$` to the final scope.
7. Verify the effective policy on `PC26` and run the final replication check.

## Configured audit policy

Only the following Advanced Audit Policy subcategories were configured in this phase:

| Category | Subcategory | Setting |
| --- | --- | --- |
| Logon/Logoff | Audit Logon | Success and Failure |
| Account Management | Audit User Account Management | Success and Failure |

The Security Option **Audit: Force audit policy subcategory settings (Windows Vista or later) to override audit policy category settings** was enabled so the advanced subcategory settings take precedence over legacy category-level settings.

No other Advanced Audit Policy subcategory is claimed as a Phase 18 configuration.

## PC27 pilot validation

`gpresult` listed `GPO - DC Advanced Auditing` under the applied computer policies on `PC27`. Effective-policy checks then returned:

```text
Logon                   Success and Failure
User Account Management Success and Failure
```

### Functional event test

A temporary, non-privileged account was created and deleted while administration targeted `PC27.virexon.local`:

| Test object | Value |
| --- | --- |
| Display name | `Audit Test User` |
| Account name | `audit.test` |
| Privileged membership | None documented |
| Final state | Deleted |

Event Viewer recorded:

| Event | Meaning | Captured details |
| ---: | --- | --- |
| `4720` | A user account was created | Target `VIREXON\audit.test`; subject `VIREXON\adm-sami.ahmed`; computer `PC27.virexon.local` |
| `4726` | A user account was deleted | Target `VIREXON\audit.test`; subject `VIREXON\adm-sami.ahmed`; computer `PC27.virexon.local` |

These events functionally validate successful User Account Management auditing. The Logon subcategory was confirmed as effective by `auditpol`, but dedicated tests for both event IDs `4624` and `4625` were not retained and are not claimed.

## Replication checkpoint and final rollout

An initial post-pilot check recorded replication error `1722`, **The RPC server is unavailable**, for part of the path from `PC26` toward `PC27`. Some naming contexts were succeeding while Configuration and Schema had recorded failures.

The rollout stopped at the pilot boundary until a later `repadmin /replsummary` returned zero failures. The evidence does not establish that the auditing GPO caused the transient RPC condition, and no unsupported root cause is assigned.

After the healthy checkpoint, Security Filtering was expanded to:

- `PC26$`
- `PC27$`

On `PC26`, `gpresult` listed the auditing GPO as applied, and `auditpol` showed both selected subcategories at **Success and Failure**.

The same `gpresult` capture lists `GPO - DC Windows Defender Firewall` as denied by Security Filtering. That is expected and confirms that the Phase 17 firewall configuration remains a `PC27`-only pilot.

The final `repadmin /replsummary` reports `0 / 5` source and destination failures for both Domain Controllers.

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [Pre-Auditing Replication](Screenshots/01-Pre-Auditing-AD-Replication-Health.png) | Both DCs report zero replication failures before the change. |
| 02 | [PC27 Audit Baseline](Screenshots/02-Pre-Auditing-PC27-Audit-Policy-Baseline.png) | Logon was Success/Failure and User Account Management was Success before the GPO. |
| 03 | [Auditing GPO Pilot Scope](Screenshots/03-DC-Auditing-GPO-Pilot-Scope.png) | The GPO is linked to the DC OU and initially filtered to `PC27$`. |
| 04 | [Advanced Audit Policy](Screenshots/04-DC-Advanced-Audit-Policy-Configuration.png) | The two selected subcategories are configured for Success and Failure. |
| 05 | [Subcategory Override](Screenshots/05-DC-Audit-Subcategory-Override-Configuration.png) | Advanced subcategory settings are configured to override legacy category settings. |
| 06 | [PC27 Policy Result](Screenshots/06-PC27-Auditing-Policy-Result.png) | The auditing GPO is applied on `PC27`. |
| 07 | [PC27 Effective Policy](Screenshots/07-PC27-Effective-Audit-Policy.png) | Both selected subcategories are effective as Success and Failure. |
| 08 | [User Creation Event](Screenshots/08-PC27-User-Creation-Audit-Event.png) | Event `4720` records creation of `audit.test` on `PC27`. |
| 09 | [User Deletion Event](Screenshots/09-PC27-User-Deletion-Audit-Event.png) | Event `4726` records deletion of `audit.test` on `PC27`. |
| 10 | [PC27 Pilot Replication](Screenshots/10-PC27-Pilot-AD-Replication-Health.png) | The retained pilot checkpoint reports zero replication failures. |
| 11 | [Final Auditing Scope](Screenshots/11-DC-Auditing-GPO-Final-Scope.png) | Security Filtering contains `PC26$` and `PC27$`. |
| 12 | [PC26 Policy Result](Screenshots/12-PC26-Auditing-Policy-Result.png) | The auditing GPO applies to `PC26` while the firewall GPO remains filtered out. |
| 13 | [PC26 Effective Policy](Screenshots/13-PC26-Effective-Audit-Policy.png) | Both selected subcategories are effective as Success and Failure on `PC26`. |
| 14 | [Final Replication Health](Screenshots/14-Final-Auditing-AD-Replication-Health.png) | Both DCs report zero final source and destination failures. |

## Scope boundaries

- Functional event validation covers successful user creation and deletion only.
- Effective Logon auditing is verified, but deliberate successful/failed logon event tests are not claimed.
- No file-system SACLs, Directory Service Changes, process creation, PowerShell logging, Sysmon, Windows Event Forwarding, SIEM integration, or centralized retention was implemented.
- No backup of `GPO - DC Advanced Auditing` is evidenced or claimed.
- A passing final replication summary is not described as proof of end-to-end monitoring coverage.

## Outcome

`GPO - DC Advanced Auditing` now applies to both Domain Controllers. Audit Logon and User Account Management are effective for Success and Failure on `PC26` and `PC27`; events `4720` and `4726` validate account creation and deletion on `PC27`; and the final replication summary reports zero failures.

**18 - Advanced Auditing — Completed ✅**
