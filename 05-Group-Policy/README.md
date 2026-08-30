# 05 - Group Policy

## Purpose

This phase implements ten Group Policy Objects (GPOs) for domain account policy, user experience and restrictions, and computer controls. The final design uses the actual `VIREXON` OU hierarchy created in [Phase 03](../03-Organizational-Units); there is no `Corporate` OU in the verified directory structure.

## Final link architecture

| GPO (deployed display name) | Configuration | Final link scope |
| --- | --- | --- |
| `GPO - Password Policy` | Computer / domain account policy | `virexon.local` domain root |
| `GPO - Account Lockout Policy` | Computer / domain account policy | `virexon.local` domain root |
| `GPO - Corporate Desktop Wallpaper` | User | `VIREXON\Users` |
| `GPO - Prevent Control Panel and Setting` | User | `Finance`, `HR`, `Marketing`, and `Sales` user OUs |
| `GPO - Disable Command Prompt` | User | `Finance`, `HR`, `Marketing`, and `Sales` user OUs |
| `GPO - Prevent Access to Registry Editor` | User | `Finance`, `HR`, `Marketing`, and `Sales` user OUs |
| `GPO - Remove Run Command` | User | `Finance`, `HR`, `Marketing`, and `Sales` user OUs |
| `GPO - Disable Task Manager` | User | `Finance`, `HR`, `Marketing`, and `Sales` user OUs |
| `GPO - Disable USB Storage` | Computer | `VIREXON\Computers` |
| `GPO - Interactive-Logon-Message` | Computer | `VIREXON\Computers` |

The wallpaper is inherited by the departmental user OUs. The five user-restriction GPOs are linked only to the four listed department OUs; `IT` and `Management` have no direct links to those restrictions. The two computer GPOs are inherited by child OUs beneath `VIREXON\Computers`.

## Domain account policies

### Password policy

The `GPO - Password Policy` screenshot records the following lab configuration:

| Setting | Configured value |
| --- | --- |
| Enforce password history | 3 passwords remembered |
| Maximum password age | 90 days |
| Minimum password age | 30 days |
| Minimum password length | 8 characters |
| Password must meet complexity requirements | Enabled |

[Screenshot 32](Screenshots/32-GPO-Password-Policy-Applied.png) shows a password-change rejection on the Windows client. It confirms that a configured password requirement affected the request, but the message does not identify which individual setting caused the rejection.

These values document the lab; they are not presented as a universal production baseline. In particular, a 30-day minimum age can prevent a user from changing a password again during that interval and therefore requires an explicit operational justification. Microsoft describes the setting and its interaction with password history in [Minimum password age](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/minimum-password-age).

The account-creation example in [Phase 04](../04-Users-and-Groups) also has **Password never expires** selected. That account-level exception prevents maximum-password-age expiration for the affected account and must be reviewed separately from the domain policy.

### Account lockout policy

| Setting | Configured value |
| --- | --- |
| Account lockout threshold | 5 invalid sign-in attempts |
| Account lockout duration | 15 minutes |
| Reset account lockout counter after | 15 minutes |
| Allow Administrator account lockout | Enabled |

[Screenshot 36](Screenshots/36-GPO-Account-Lockout-Policy-Applied.png) is functional evidence: the client displays that the referenced account is locked out. This replaces the previous, inaccurate claim that the screenshot was `net accounts` output.

Allowing lockout of the built-in Administrator account changes the recovery and availability risk of the lab. A production adoption would require a tested administrative recovery path and an organization-approved lockout standard.

## User policies

| Policy | Intended behavior | Scope |
| --- | --- | --- |
| Corporate desktop wallpaper | Applies the VIREXON wallpaper to user sessions. | `VIREXON\Users` and inherited child OUs |
| Prevent Control Panel and Setting | Blocks Control Panel and Windows Settings entry points. | Finance, HR, Marketing, Sales |
| Disable Command Prompt | Prevents use of Command Prompt for the targeted users. | Finance, HR, Marketing, Sales |
| Prevent Access to Registry Editor | Prevents the targeted users from opening Registry Editor. | Finance, HR, Marketing, Sales |
| Remove Run Command | Removes the Run command from the targeted user interface. | Finance, HR, Marketing, Sales |
| Disable Task Manager | Prevents the targeted users from opening Task Manager. | Finance, HR, Marketing, Sales |

The GPO names above reproduce the deployed display names exactly, including the singular word `Setting` in `GPO - Prevent Control Panel and Setting`.

## Computer policies

| Policy | Configured behavior | Scope |
| --- | --- | --- |
| `GPO - Disable USB Storage` | **All Removable Storage classes: Deny all access** is enabled. | `VIREXON\Computers` |
| `GPO - Interactive-Logon-Message` | Displays an authorization notice before sign-in. | `VIREXON\Computers` |

The logon notice uses the title **Authorized Access Only** and the following message:

> This system is the property of VIREXON.
>
> Unauthorized access is prohibited.
>
> All activities may be monitored and recorded.

The removable-storage test shows `E:\` returning **Access is denied** after policy refresh. The logon-message test shows the configured notice before authentication.

## Scope validation

The final Group Policy Management captures establish that:

- Password and lockout GPOs are linked at the `virexon.local` domain root.
- The actual custom root OU is `VIREXON`, with separate `Users` and `Computers` branches.
- The wallpaper is linked at `VIREXON\Users`.
- The five restriction GPOs are linked beneath Finance, HR, Marketing, and Sales—not IT or Management.
- USB-storage and interactive-logon GPOs are linked at `VIREXON\Computers`.
- The parent-scope screenshots show the links enabled and not marked **Enforced**.

### Evidence discrepancy: screenshots 45 and 46

The visible outcomes in the last two screenshots are the reverse of their filenames:

- [Screenshot 45](Screenshots/45-GPO-Validation-HR-Restrictions-Applied.png) visibly shows Control Panel and Run available.
- [Screenshot 46](Screenshots/46-GPO-Validation-IT-Restrictions-Excluded.png) visibly shows a restriction-block message.

Neither screenshot displays the signed-in identity. They prove that both unrestricted and restricted client behaviors were captured, but they do **not** independently prove which session was HR and which was IT. The filenames are therefore treated as legacy metadata, not as sufficient identity evidence. A future evidence refresh should capture `whoami` or `gpresult /r` together with the result in one screenshot per test session.

## Evidence index

### Configuration and behavior

| GPO | Configuration evidence | Client or result evidence |
| --- | --- | --- |
| Corporate Desktop Wallpaper | [03 - Configured](Screenshots/03-GPO-Corporate-Desktop-Wallpaper-Configured.png) | [04 - Applied](Screenshots/04-GPO-Corporate-Desktop-Wallpaper-Applied.png) |
| Prevent Control Panel and Setting | [07 - Configured](Screenshots/07-GPO-Prevent-Control-Panel-Configured.png) | [08 - Applied](Screenshots/08-GPO-Prevent-Control-Panel-Applied.png) |
| Disable Command Prompt | [11 - Configured](Screenshots/11-GPO-Disable-Command-Prompt-Configured.png) | [12 - Applied](Screenshots/12-GPO-Disable-Command-Prompt-Applied.png) |
| Prevent Access to Registry Editor | [15 - Configured](Screenshots/15-GPO-Disable-Registry-Editor-Configured.png) | [16 - Applied](Screenshots/16-GPO-Disable-Registry-Editor-Applied.png) |
| Remove Run Command | [19 - Configured](Screenshots/19-GPO-Remove-Run-Command-Configured.png) | [20 - Applied](Screenshots/20-GPO-Remove-Run-Command-Applied.png) |
| Disable Task Manager | [23 - Configured](Screenshots/23-GPO-Disable-Task-Manager-Configured.png) | [24 - Applied](Screenshots/24-GPO-Disable-Task-Manager-Applied.png) |
| Disable USB Storage | [27 - Configured](Screenshots/27-GPO-Disable-USB-Storage-Configured.png) | [28 - Access Denied](Screenshots/28-GPO-Disable-USB-Storage-Applied.png) |
| Password Policy | [31 - Configured](Screenshots/31-GPO-Password-Policy-Configured.png) | [32 - Password Rejected](Screenshots/32-GPO-Password-Policy-Applied.png) |
| Account Lockout Policy | [35 - Configured](Screenshots/35-GPO-Account-Lockout-Policy-Configured.png) | [36 - Account Locked](Screenshots/36-GPO-Account-Lockout-Policy-Applied.png) |
| Interactive Logon Message | [39 - Configured](Screenshots/39-GPO-Interactive-Logon-Message-Configured.png) | [40 - Displayed](Screenshots/40-GPO-Interactive-Logon-Message-Applied.png) |

### Final architecture

| Evidence | Verified state |
| --- | --- |
| [41 - Domain Root Links](Screenshots/41-GPO-Final-Domain-Root-Linking.png) | Password and account-lockout links at `virexon.local`. |
| [42 - VIREXON OU Architecture](Screenshots/42-GPO-Final-Corporate-OU-Architecture.png) | Actual `VIREXON` root with Users and Computers scopes. |
| [43 - User GPO Scope](Screenshots/43-GPO-Final-Users-GPO-Scope.png) | Wallpaper parent link and restriction links on four department OUs. |
| [44 - Computer GPO Scope](Screenshots/44-GPO-Final-Computers-GPO-Scope.png) | USB and interactive-logon links at `VIREXON\Computers`. |
| [45 - Controls Available](Screenshots/45-GPO-Validation-HR-Restrictions-Applied.png) | Visible unrestricted behavior; identity is not visible. |
| [46 - Restriction Message](Screenshots/46-GPO-Validation-IT-Restrictions-Excluded.png) | Visible restricted behavior; identity is not visible. |

<details>
<summary>Creation and initial-link screenshots</summary>

| GPO | Created | Initially linked |
| --- | --- | --- |
| Corporate Desktop Wallpaper | [01](Screenshots/01-GPO-Corporate-Desktop-Wallpaper-Created.png) | [02](Screenshots/02-GPO-Corporate-Desktop-Wallpaper-Linked.png) |
| Prevent Control Panel and Setting | [05](Screenshots/05-GPO-Prevent-Control-Panel-Created.png) | [06](Screenshots/06-GPO-Prevent-Control-Panel-Linked.png) |
| Disable Command Prompt | [09](Screenshots/09-GPO-Disable-Command-Prompt-Created.png) | [10](Screenshots/10-GPO-Disable-Command-Prompt-Linked.png) |
| Prevent Access to Registry Editor | [13](Screenshots/13-GPO-Disable-Registry-Editor-Created.png) | [14](Screenshots/14-GPO-Disable-Registry-Editor-Linked.png) |
| Remove Run Command | [17](Screenshots/17-GPO-Remove-Run-Command-Created.png) | [18](Screenshots/18-GPO-Remove-Run-Command-Linked.png) |
| Disable Task Manager | [21](Screenshots/21-GPO-Disable-Task-Manager-Created.png) | [22](Screenshots/22-GPO-Disable-Task-Manager-Linked.png) |
| Disable USB Storage | [25](Screenshots/25-GPO-Disable-USB-Storage-Created.png) | [26](Screenshots/26-GPO-Disable-USB-Storage-Linked.png) |
| Password Policy | [29](Screenshots/29-GPO-Password-Policy-Created.png) | [30](Screenshots/30-GPO-Password-Policy-Linked.png) |
| Account Lockout Policy | [33](Screenshots/33-GPO-Account-Lockout-Policy-Created.png) | [34](Screenshots/34-GPO-Account-Lockout-Policy-Linked.png) |
| Interactive Logon Message | [37](Screenshots/37-GPO-Interactive-Logon-Message-Created.png) | [38](Screenshots/38-GPO-Interactive-Logon-Message-Linked.png) |

</details>

## Outcome

The final design contains ten documented GPOs with explicit domain, user, and computer scopes. Configuration evidence exists for every policy, and client-side behavior is captured for each control. The remaining validation limitation is narrowly defined: the final restricted and unrestricted sessions need identity-bearing evidence before they can be attributed to HR and IT with confidence.
