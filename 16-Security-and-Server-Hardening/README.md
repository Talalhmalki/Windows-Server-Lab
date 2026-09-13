# 16 - Security and Server Hardening

## Purpose

This phase applies a controlled set of security-hardening settings to the two VIREXON Domain Controllers. Deployment begins with `PC27` as the pilot and expands to `PC26` only after policy application, service state, sign-in behavior, and replication have been validated.

## Verified environment

| System | Address | Function |
| --- | --- | --- |
| `PC26.virexon.local` | `192.168.1.2` | Primary DC, DNS, DHCP, file services, and final FSMO role holder |
| `PC27.virexon.local` | `192.168.1.3` | Additional writable DC, DNS server, and Global Catalog |
| `PC-IT-01` | `192.168.1.50` | Domain client and RSAT administrative workstation |

| Directory component | Value |
| --- | --- |
| Domain | `virexon.local` |
| Active Directory site | `Riyadh-HQ` |
| Network | `192.168.1.0/24` |
| Virtualization | VMware Workstation Pro |
| Evidence | 16 screenshots |

## Deployment design

The dedicated GPO retained the name:

```text
GPO - DC Disable Print Spooler
```

Although its name highlights the Print Spooler control, the captured settings report shows that this GPO contains the full Phase 16 hardening set. The name is documented as implemented; no repository text implies that it contains only one setting.

The GPO was linked to the `Domain Controllers` OU. Security Filtering initially contained only `PC27$`. After the pilot passed, `PC26$` was added so the final scope contained both Domain Controllers.

## Configured controls

| Area | Group Policy setting | Configured value |
| --- | --- | --- |
| Sign-in privacy | Interactive logon: Don't display last signed-in | Enabled |
| SMB client | Digitally sign communications (always) | Enabled |
| SMB server | Digitally sign communications (always) | Enabled |
| Legacy authentication | LAN Manager authentication level | Send NTLMv2 response only; refuse LM and NTLM |
| System service | Print Spooler startup mode | Disabled |

The Print Spooler setting reduces unnecessary service exposure on Domain Controllers that do not provide print services. SMB1 was checked separately and observed as disabled on both servers; the retained GPO report does not show a Phase 16 setting that disabled SMB1.

## PC27 pilot validation

After Group Policy refresh, `gpresult /r /scope computer` listed `GPO - DC Disable Print Spooler` under the applied computer policies on `PC27`.

The operational checks showed:

- Print Spooler startup type `DISABLED`;
- Print Spooler state `STOPPED`;
- a manual start blocked with system error `1058`;
- SMB1 feature state `Disabled`; and
- a sign-in screen displaying **Other user** instead of the previous username.

The pilot `repadmin /replsummary` result showed `0 / 5` source and destination failures for both Domain Controllers.

## Final rollout to PC26

After `PC26$` was added to Security Filtering, `gpresult` confirmed that the same hardening GPO applied to `PC26`. Its service and sign-in checks matched the pilot:

- Print Spooler disabled and stopped;
- manual startup blocked with error `1058`;
- SMB1 disabled; and
- the previous signed-in identity not displayed.

The screenshots prove effective GPO application and the selected observable controls. They do not constitute a complete security baseline assessment of every local or domain setting.

## Service-continuity validation

Post-hardening checks were performed from `PC-IT-01` and the Domain Controllers:

| Check | Captured result |
| --- | --- |
| Client addressing | `PC-IT-01` retained `192.168.1.50` with DHCP enabled |
| DNS | A query for `virexon.local` succeeded through `PC26` and returned both DC addresses |
| Client GPO scope | The DC hardening GPO is absent from the client's applied policy list |
| Department share | The IT department path beneath `\\192.168.1.2\Departments` is accessible |
| RSAT | Active Directory Users and Computers opens the domain remotely |
| DC services | `NTDS`, `DNS`, `Netlogon`, and `DFSR` report `RUNNING` on both DCs |
| Final replication | Both DCs report `0 / 5` source and destination failures |

The client lease and successful name lookup demonstrate the captured client state at validation time; they are not presented as a separate DHCP high-availability test.

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [Pre-Hardening Replication](Screenshots/01-Pre-Hardening-AD-Replication-Health.png) | Both DCs report zero replication failures before the change. |
| 02 | [Pilot GPO Scope](Screenshots/02-DC-Hardening-GPO-Pilot-Scope.png) | The GPO is linked to the DC OU and initially filtered to `PC27$`. |
| 03 | [Security Options](Screenshots/03-DC-Hardening-Security-Options.png) | Sign-in privacy, SMB signing, NTLMv2, and Spooler settings are configured. |
| 04 | [Print Spooler Policy](Screenshots/04-DC-Hardening-Print-Spooler-Policy.png) | Print Spooler startup is explicitly set to Disabled. |
| 05 | [PC27 Policy Result](Screenshots/05-PC27-Hardening-Policy-Result.png) | The hardening GPO is applied to `PC27`. |
| 06 | [PC27 Service and SMB1](Screenshots/06-PC27-Service-and-SMB1-Verification.png) | Spooler is disabled/stopped, startup is blocked, and SMB1 is disabled. |
| 07 | [PC27 Sign-In Protection](Screenshots/07-PC27-Sign-In-Protection-Verification.png) | The last signed-in identity is not displayed. |
| 08 | [PC27 Pilot Replication](Screenshots/08-PC27-Pilot-AD-Replication-Health.png) | Replication remains at zero failures after the pilot. |
| 09 | [Final GPO Scope](Screenshots/09-DC-Hardening-GPO-Final-Scope.png) | Security Filtering contains `PC26$` and `PC27$`. |
| 10 | [PC26 Policy Result](Screenshots/10-PC26-Hardening-Policy-Result.png) | The hardening GPO is applied to `PC26`. |
| 11 | [PC26 Service and SMB1](Screenshots/11-PC26-Service-and-SMB1-Verification.png) | The same Spooler and SMB1 states are present on `PC26`. |
| 12 | [PC26 Sign-In Protection](Screenshots/12-PC26-Sign-In-Protection-Verification.png) | The last signed-in identity is not displayed on `PC26`. |
| 13 | [Client DNS and DHCP State](Screenshots/13-Client-DNS-and-DHCP-Verification.png) | The client retains its DHCP address and successfully resolves the domain. |
| 14 | [Client File Access and GPO Scope](Screenshots/14-Client-File-Access-and-GPO-Verification.png) | Department data remains accessible and the DC GPO is not applied to the client. |
| 15 | [RSAT Administration](Screenshots/15-RSAT-Remote-AD-Administration-Verification.png) | Remote directory administration remains available. |
| 16 | [Final Services and Replication](Screenshots/16-Final-DC-Services-and-Replication-Health.png) | Core services run on both DCs and final replication reports zero failures. |

## Rollback and operational considerations

- The settings reside in a dedicated GPO rather than either default domain policy, providing a focused rollback boundary.
- A problematic setting should be reverted within the dedicated GPO, followed by policy refresh, service validation, and replication checks.
- Disabling Print Spooler is appropriate here because neither DC is documented as a print server.
- Requiring SMB signing and refusing LM/NTLM should be compatibility-tested against legacy systems before production deployment.
- This phase implements a selected hardening set, not the complete Microsoft security baseline or an external compliance benchmark.

## Outcome

The selected hardening controls were deployed first to `PC27` and then to `PC26`. Both servers show the intended GPO, disabled Print Spooler, disabled SMB1 state, and protected sign-in display. Client administration and file access remained available, core DC services remained running, and final replication reported zero failures.

**16 - Security and Server Hardening — Completed ✅**
