# 17 - Windows Defender Firewall

## Purpose

This phase introduces centrally managed Windows Defender Firewall settings for Domain Controllers through a controlled pilot on `PC27`. The policy standardizes the Domain Profile, preserves existing local rule merging, records dropped traffic, and permits management ICMP from the designated administrative workstation.

The phase intentionally ends as a **PC27-only pilot**. No deployment to `PC26` is claimed.

## Verified environment

| Component | Configuration |
| --- | --- |
| Domain | `virexon.local` |
| Active Directory site | `Riyadh-HQ` |
| Primary Domain Controller | `PC26.virexon.local` — `192.168.1.2` |
| Pilot Domain Controller | `PC27.virexon.local` — `192.168.1.3` |
| Management workstation | `PC-IT-01` — `192.168.1.50` |
| Firewall management | Group Policy |
| Evidence | 9 screenshots |

## Baseline and pilot scope

The pre-change `repadmin /replsummary` capture reports `0 / 5` source and destination failures for both DCs.

Before the GPO was applied, `PC27` already showed:

- Windows Defender Firewall enabled;
- unmatched inbound connections blocked; and
- unmatched outbound connections allowed.

The phase therefore centralizes and verifies existing desired behavior; it does not claim that the firewall was previously disabled.

The dedicated GPO is:

```text
GPO - DC Windows Defender Firewall
```

It is linked to the `Domain Controllers` OU, with Security Filtering limited to `PC27$` for the pilot.

## Domain Profile configuration

| Setting | Configured value |
| --- | --- |
| Firewall state | On |
| Default inbound connections | Block |
| Default outbound connections | Allow |
| Apply local firewall rules | Yes |
| Log dropped packets | Yes |
| Log successful connections | No |

Private and Public profiles were not configured by this GPO. Allowing local rule merging preserves the existing Windows and server-role rules rather than attempting to recreate the complete AD DS, DNS, SMB, RPC, and DFSR rule set manually.

## Management ICMP rule

The dedicated inbound rule is named:

```text
VIREXON - Allow ICMPv4 Echo from PC-IT-01
```

| Property | Captured value |
| --- | --- |
| Action | Allow |
| Protocol | ICMPv4 |
| Remote address | `192.168.1.50` |
| Profile | Domain |
| Enabled | Yes |

The source address belongs to `PC-IT-01`. The rule is narrower than permitting ICMP from the entire subnet.

## Pilot validation

After `gpupdate /force`, `gpresult /r /scope computer` listed `GPO - DC Windows Defender Firewall` as applied on `PC27`.

The effective firewall console then showed the active Domain Profile under Group Policy control, with unmatched inbound traffic blocked, outbound traffic allowed, and dropped-packet logging enabled.

From `PC-IT-01`:

- `hostname` identified the source workstation;
- `whoami` identified `VIREXON\adm-sami.ahmed`;
- `ipconfig` showed `192.168.1.50`; and
- four ICMP Echo requests to `192.168.1.3` received four replies with zero loss.

Because local firewall rule merging remains enabled, the successful ping proves approved management reachability from `PC-IT-01`. It does not independently prove that the custom rule was the only rule capable of permitting that traffic.

The post-change `repadmin /replsummary` capture again reports zero source and destination failures for both DCs.

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [Pre-Firewall Replication](Screenshots/01-Pre-Firewall-AD-Replication-Health.png) | Both DCs report zero replication failures before the pilot. |
| 02 | [PC27 Firewall Baseline](Screenshots/02-Pre-Firewall-PC27-Profile-State.png) | Firewall-on, inbound-block, and outbound-allow behavior existed before the GPO. |
| 03 | [Firewall GPO Pilot Scope](Screenshots/03-DC-Firewall-GPO-Pilot-Scope.png) | The GPO is linked to the DC OU and filtered only to `PC27$`. |
| 04 | [Domain Profile Configuration](Screenshots/04-DC-Firewall-Profile-Configuration.png) | Domain Profile defaults, rule merging, and logging settings are configured. |
| 05 | [Management ICMP Rule](Screenshots/05-DC-Firewall-Management-ICMP-Rule.png) | The enabled Domain-profile ICMPv4 allow rule is scoped to `192.168.1.50`. |
| 06 | [PC27 Policy Result](Screenshots/06-PC27-Firewall-Policy-Result.png) | The dedicated firewall GPO is applied on `PC27`. |
| 07 | [PC27 Effective State](Screenshots/07-PC27-Firewall-Effective-State.png) | The effective Domain Profile is controlled by Group Policy with dropped-packet logging. |
| 08 | [Management ICMP Test](Screenshots/08-PC27-Management-ICMP-Verification.png) | `PC-IT-01` at `192.168.1.50` reaches `PC27` with zero packet loss. |
| 09 | [Post-Firewall Replication](Screenshots/09-PC27-Post-Firewall-AD-Replication-Health.png) | Replication remains at zero failures after the pilot. |

## Scope boundaries

- `PC26` was not added to this firewall GPO.
- The test did not attempt to replace the built-in Windows server-role rules.
- No unused-port negative test or firewall-log event correlation was retained.
- No IPsec or Connection Security Rules were configured.
- The phase validates the Domain Profile and one approved management path, not every AD DS port or client workflow.
- Stopping the Windows Defender Firewall service is not an appropriate rollback method. The dedicated GPO scope or individual settings provide the controlled rollback path.

Phase 18 later confirms the same boundary: `PC26` reports this firewall GPO as denied by Security Filtering.


## Screenshot evidence

The screenshots below follow the documented evidence order. Each image links to its original file.

### 01 - Pre-Firewall Replication

[![01 - Pre-Firewall Replication](Screenshots/01-Pre-Firewall-AD-Replication-Health.png)](Screenshots/01-Pre-Firewall-AD-Replication-Health.png)

### 02 - PC27 Firewall Baseline

[![02 - PC27 Firewall Baseline](Screenshots/02-Pre-Firewall-PC27-Profile-State.png)](Screenshots/02-Pre-Firewall-PC27-Profile-State.png)

### 03 - Firewall GPO Pilot Scope

[![03 - Firewall GPO Pilot Scope](Screenshots/03-DC-Firewall-GPO-Pilot-Scope.png)](Screenshots/03-DC-Firewall-GPO-Pilot-Scope.png)

### 04 - Domain Profile Configuration

[![04 - Domain Profile Configuration](Screenshots/04-DC-Firewall-Profile-Configuration.png)](Screenshots/04-DC-Firewall-Profile-Configuration.png)

### 05 - Management ICMP Rule

[![05 - Management ICMP Rule](Screenshots/05-DC-Firewall-Management-ICMP-Rule.png)](Screenshots/05-DC-Firewall-Management-ICMP-Rule.png)

### 06 - PC27 Policy Result

[![06 - PC27 Policy Result](Screenshots/06-PC27-Firewall-Policy-Result.png)](Screenshots/06-PC27-Firewall-Policy-Result.png)

### 07 - PC27 Effective State

[![07 - PC27 Effective State](Screenshots/07-PC27-Firewall-Effective-State.png)](Screenshots/07-PC27-Firewall-Effective-State.png)

### 08 - Management ICMP Test

[![08 - Management ICMP Test](Screenshots/08-PC27-Management-ICMP-Verification.png)](Screenshots/08-PC27-Management-ICMP-Verification.png)

### 09 - Post-Firewall Replication

[![09 - Post-Firewall Replication](Screenshots/09-PC27-Post-Firewall-AD-Replication-Health.png)](Screenshots/09-PC27-Post-Firewall-AD-Replication-Health.png)

## Outcome

`PC27` now has a centrally managed Domain Profile pilot with inbound traffic blocked by default, outbound traffic allowed, local rule merging retained, and dropped-packet logging enabled. The scoped management workstation remained reachable, and Active Directory replication remained healthy. The configuration is complete for the documented pilot scope only.

**17 - Windows Defender Firewall — Completed as a controlled pilot ✅**
