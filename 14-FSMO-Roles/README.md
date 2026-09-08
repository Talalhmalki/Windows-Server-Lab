# 14 - FSMO Roles

## Overview

This phase demonstrates the identification, authorization, planned transfer, verification, and restoration of the five Flexible Single Master Operations (**FSMO**) roles in the `virexon.local` Active Directory environment.

The lab contains two writable Domain Controllers:

- `PC26` — the original FSMO role holder.
- `PC27` — an additional writable Domain Controller used as the temporary transfer target.

All five roles were transferred from `PC26` to `PC27` through the native Active Directory graphical management tools. The transfer was verified from the command line, and the roles were then returned to `PC26` to restore the original lab design.

Replication health was validated before and after the operation.

| Item | Value |
|---|---|
| Status | Completed |
| Active Directory Domain | `virexon.local` |
| NetBIOS Domain Name | `VIREXON` |
| Active Directory Site | `Riyadh-HQ` |
| Original FSMO Role Holder | `PC26.virexon.local` |
| Temporary FSMO Role Holder | `PC27.virexon.local` |
| Final FSMO Role Holder | `PC26.virexon.local` |
| Transfer Method | Native Active Directory GUI tools |
| Verification Tools | `netdom`, `repadmin`, and `whoami` |
| Evidence | 6 screenshots |

## Objectives

- Identify the current owners of all five FSMO roles.
- Understand the purpose and scope of each FSMO role.
- Validate Active Directory replication before transferring any role.
- Verify that the administrative account has the required authorization.
- Register and use the Active Directory Schema MMC snap-in.
- Perform a planned transfer of all five FSMO roles from `PC26` to `PC27`.
- Verify that `PC27` became the temporary owner of all five roles.
- Return all five roles to `PC26`.
- Confirm the final FSMO role ownership.
- Validate replication health after the completed transfer cycle.
- Distinguish a planned role transfer from an emergency role seizure.

## Lab Environment

| System | IPv4 Address | Function |
|---|---|---|
| `PC26` | `192.168.1.2/24` | Existing writable Domain Controller, DNS server, DHCP server, and final FSMO role holder |
| `PC27` | `192.168.1.3/24` | Additional writable Domain Controller, DNS server, Global Catalog, and temporary FSMO transfer target |

Both Domain Controllers belong to the `virexon.local` domain and reside in the `Riyadh-HQ` Active Directory site.

The transfer was performed while both Domain Controllers were online and able to replicate. No client computer was required for this phase.

---

## FSMO Role Architecture

Active Directory normally supports multi-master updates, allowing writable Domain Controllers to process directory changes. However, certain operations require a single authoritative Domain Controller to prevent conflicting changes.

These operations are assigned through five FSMO roles.

A FSMO role is not installed independently on every Domain Controller. Each role has exactly one owner at a time within its applicable forest or domain scope.

### Forest-Wide Roles

| FSMO Role | Responsibility |
|---|---|
| **Schema Master** | Controls modifications to the Active Directory schema, including changes to object classes and attributes. Only one Schema Master exists in the forest. |
| **Domain Naming Master** | Controls the addition and removal of domains and application directory partitions within the forest. Only one Domain Naming Master exists in the forest. |

### Domain-Wide Roles

| FSMO Role | Responsibility |
|---|---|
| **PDC Emulator** | Provides priority handling for password changes, participates in account lockout processing, serves as the preferred Group Policy administration target, and acts as the authoritative time source for the forest when located in the forest root domain. |
| **RID Master** | Allocates Relative Identifier pools to Domain Controllers so they can create users, groups, and computers with unique Security Identifiers. |
| **Infrastructure Master** | Maintains references to objects located in other domains. Its operational impact is limited in this single-domain environment, but it must still have a valid role owner. |

Because `virexon.local` contains one domain, the environment has two forest-wide roles and three domain-wide roles, for a total of five FSMO roles.

FSMO ownership does not automatically move when a Domain Controller is shut down. Another writable Domain Controller can continue providing many Active Directory services without automatically becoming the FSMO role holder.

---

## FSMO Management Tools

The five roles are managed through three native Active Directory graphical tools.

| Management Tool | FSMO Roles Managed |
|---|---|
| Active Directory Users and Computers | RID Master, PDC Emulator, and Infrastructure Master |
| Active Directory Domains and Trusts | Domain Naming Master |
| Active Directory Schema MMC Snap-in | Schema Master |

The Active Directory Schema management snap-in was registered before it was added to Microsoft Management Console:

```cmd
regsvr32 schmmgmt.dll
```

Registering this snap-in makes the Schema management interface available. It does not enable or modify the Active Directory schema itself.

---

## Planned Transfer Design

The transfer was defined as a temporary administrative exercise rather than a permanent redesign of FSMO role placement.

| Change Item | Decision |
|---|---|
| Source Domain Controller | `PC26.virexon.local` |
| Temporary Target Domain Controller | `PC27.virexon.local` |
| Selected Roles | All five FSMO roles |
| Transfer Type | Planned and graceful transfer |
| Administrative Method | Native GUI management tools |
| Intermediate State | All five roles owned by `PC27` |
| Final State | All five roles returned to `PC26` |
| Seizure | Not performed |
| Domain Controller Demotion | Not performed |

This design demonstrated the complete transfer process while preserving `PC26` as the final FSMO role holder for the existing lab configuration.

---

## 01 - Current FSMO Role Ownership

The initial FSMO ownership was identified with:

```cmd
netdom query fsmo
```

The query confirmed that `PC26.virexon.local` owned all five roles before the transfer.

| FSMO Role | Initial Owner |
|---|---|
| Schema Master | `PC26.virexon.local` |
| Domain Naming Master | `PC26.virexon.local` |
| PDC Emulator | `PC26.virexon.local` |
| RID Master | `PC26.virexon.local` |
| Infrastructure Master | `PC26.virexon.local` |

This established the authoritative baseline without assuming role placement from the Domain Controller deployment order.

![Current FSMO Role Ownership](Screenshots/01-Current-FSMO-Role-Ownership.png)

---

## 02 - Pre-Transfer Replication Health

Replication health was checked before transferring any FSMO role:

```cmd
repadmin /replsummary
```

The successful readiness result showed both `PC26` and `PC27` as replication sources and destinations with zero failures.

| Domain Controller | Source Failures | Destination Failures |
|---|---:|---:|
| `PC26` | `0 / 5` | `0 / 5` |
| `PC27` | `0 / 5` | `0 / 5` |

An earlier readiness attempt reported a single DNS lookup-related replication failure for the Schema directory partition from `PC26` to `PC27`. The transfer was paused, the affected replication operation was retried after both Domain Controllers were fully available, and the health check was repeated.

The retained pre-transfer result confirmed zero failures before the FSMO transfer began. No unverified root cause was assigned to the earlier transient result.

![Pre-Transfer AD Replication Health](Screenshots/02-Pre-Transfer-AD-Replication-Health.png)

---

## 03 - Transfer Authorization Verification

The administrative session used for the FSMO transfer was verified with:

```cmd
whoami
whoami /groups | findstr /I /C:"Domain Admins" /C:"Enterprise Admins" /C:"Schema Admins"
```

The active account was confirmed as:

```text
VIREXON\Administrator
```

Its security token included the required privileged groups:

| Administrative Group | Transfer Authority |
|---|---|
| `Domain Admins` | PDC Emulator, RID Master, and Infrastructure Master |
| `Enterprise Admins` | Domain Naming Master and forest-level administration |
| `Schema Admins` | Schema Master administration |

This verification confirmed that the session had the necessary authorization to transfer all five roles.

![FSMO Transfer Authorization Verification](Screenshots/03-FSMO-Transfer-Authorization-Verification.png)

---

## 04 - FSMO Role Transfer to PC27

All five roles were transferred from `PC26` to `PC27` using the native graphical management tools.

### Domain-Wide Roles

The following roles were transferred through **Active Directory Users and Computers** after connecting the console to `PC27`:

- RID Master
- PDC Emulator
- Infrastructure Master

### Domain Naming Master

The Domain Naming Master role was transferred through **Active Directory Domains and Trusts**, with `PC27` selected as the target Domain Controller.

### Schema Master

The Schema Master role was transferred through the registered **Active Directory Schema** MMC snap-in, with `PC27` selected as the target.

This was a planned transfer while the existing role holder remained online. No role seizure was used.

After the GUI operations were completed, role ownership was verified with:

```cmd
netdom query fsmo
```

The result showed that all five roles were owned by `PC27.virexon.local`.

| FSMO Role | Temporary Owner |
|---|---|
| Schema Master | `PC27.virexon.local` |
| Domain Naming Master | `PC27.virexon.local` |
| PDC Emulator | `PC27.virexon.local` |
| RID Master | `PC27.virexon.local` |
| Infrastructure Master | `PC27.virexon.local` |

**Result:** The planned transfer of all five FSMO roles from `PC26` to `PC27` completed successfully.

![Selected FSMO Role Transfer](Screenshots/04-Selected-FSMO-Role-Transfer.png)

---

## 05 - Final FSMO Role Ownership

After the temporary transfer was verified, the same graphical management tools were used to return all five roles from `PC27` to `PC26`.

A final ownership query was performed:

```cmd
netdom query fsmo
```

The retained evidence shows the ownership transition from `PC27` back to `PC26` in the same command session.

| FSMO Role | Final Owner |
|---|---|
| Schema Master | `PC26.virexon.local` |
| Domain Naming Master | `PC26.virexon.local` |
| PDC Emulator | `PC26.virexon.local` |
| RID Master | `PC26.virexon.local` |
| Infrastructure Master | `PC26.virexon.local` |

**Result:** The original FSMO placement was successfully restored, with `PC26` again owning all five roles.

![Final FSMO Role Ownership Verification](Screenshots/05-Final-FSMO-Role-Ownership-Verification.png)

---

## 06 - Post-Transfer Replication Health

Final validation confirmed both FSMO ownership and replication health after the complete transfer-and-return cycle.

The verification commands were:

```cmd
netdom query fsmo
repadmin /replsummary
```

The final results confirmed:

- All five FSMO roles were owned by `PC26`.
- `PC26` and `PC27` remained visible as replication sources and destinations.
- Both Domain Controllers reported `0 / 5` replication failures.
- No replication error was displayed.

| Verification Item | Final Result |
|---|---|
| Final FSMO Role Holder | `PC26.virexon.local` |
| `PC26` Source Replication Failures | `0 / 5` |
| `PC27` Source Replication Failures | `0 / 5` |
| `PC26` Destination Replication Failures | `0 / 5` |
| `PC27` Destination Replication Failures | `0 / 5` |
| Overall Replication Status | Healthy |

**Result:** The FSMO transfer cycle completed without leaving replication failures or an unintended role placement.

![Post-Transfer AD Replication Health](Screenshots/06-Post-Transfer-AD-Replication-Health.png)

---

## Transfer vs. Seizure

FSMO roles can be moved by either transfer or seizure, but the two operations serve different purposes.

| Comparison | Transfer | Seizure |
|---|---|---|
| Intended Use | Planned administration or maintenance | Emergency recovery |
| Existing Role Holder | Online and reachable | Permanently unavailable or unrecoverable |
| Operation | Graceful handover between Domain Controllers | Forced assignment to another Domain Controller |
| Replication Requirement | Healthy communication between the Domain Controllers | Used when normal communication and transfer are impossible |
| Former Role Holder | Can remain in service | Must not be reintroduced without appropriate recovery, cleanup, or rebuilding |
| Used in This Phase | Yes | No |

Only planned transfer operations were performed in this phase.

Seizure was intentionally excluded because both Domain Controllers were operational. No Domain Controller was forcibly removed, demoted, or subjected to metadata cleanup.

---

## Role Ownership Timeline

| Stage | Schema | Domain Naming | PDC | RID | Infrastructure |
|---|---|---|---|---|---|
| Initial State | `PC26` | `PC26` | `PC26` | `PC26` | `PC26` |
| Temporary Transfer State | `PC27` | `PC27` | `PC27` | `PC27` | `PC27` |
| Final State | `PC26` | `PC26` | `PC26` | `PC26` | `PC26` |

---

## Validation Summary

| Validation | Expected Result | Observed Result |
|---|---|---|
| Initial role discovery | Identify all five current owners | All five roles were owned by `PC26` |
| Pre-transfer replication | Zero replication failures | Passed |
| Administrative authorization | Required privileged groups present | Passed |
| Transfer to `PC27` | All five roles assigned to `PC27`27` | Passed |
| Return to `PC26` | All five roles restored to `PC26` | Passed |
| Post-transfer replication | Zero replication failures | Passed |
| Final configuration | Original role placement restored | Passed |

## Evidence Summary

| Screenshot | Evidence |
|---|---|
| `01-Current-FSMO-Role-Ownership.png` | Initial ownership of all five roles by `PC26` |
| `02-Pre-Transfer-AD-Replication-Health.png` | Healthy replication before the transfer |
| `03-FSMO-Transfer-Authorization-Verification.png` | Administrative identity and required privileged group memberships |
| `04-Selected-FSMO-Role-Transfer.png` | Successful temporary ownership of all five roles by `PC27` |
| `05-Final-FSMO-Role-Ownership-Verification.png` | Successful return of all five roles to `PC26` |
| `06-Post-Transfer-AD-Replication-Health.png` | Final FSMO ownership and healthy replication |

---

## Scope and Design Boundaries

This phase focused on planned FSMO administration between two healthy writable Domain Controllers.

The following items were intentionally outside the implementation scope:

- FSMO role seizure.
- Forced Domain Controller demotion.
- Active Directory metadata cleanup.
- Domain Controller removal.
- Active Directory schema modification.
- Creation or removal of an Active Directory domain.
- RID exhaustion or RID pool recovery testing.
- Permanent distribution of FSMO roles between the two Domain Controllers.
- Clustering or service-level high availability.

`PC27` remained an additional writable Domain Controller after the exercise. It was not configured as a passive backup server.

The temporary transfer demonstrated administrative control of the FSMO roles. The final placement on `PC26` preserved the established lab design and did not represent automatic failover behavior.

## Conclusion

The five FSMO roles in the VIREXON Active Directory environment were successfully identified, transferred, verified, and restored.

Initial ownership was confirmed on `PC26`, and replication health was validated before the change. The administrative account was verified as having the required Domain Admins, Enterprise Admins, and Schema Admins memberships.

All five roles were then transferred through the native Active Directory GUI tools to `PC27`. Command-line verification confirmed that `PC27` had become the temporary owner of every role.

The roles were subsequently returned to `PC26`, restoring the original lab design. Final `netdom` and `repadmin` results confirmed the intended ownership and zero replication failures between the two Domain Controllers.

The six screenshots provide evidence of the complete planned FSMO transfer lifecycle without performing seizure, demotion, or destructive recovery operations.

## References

- [Microsoft Learn — Flexible Single Master Operations Roles](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-fsmo-roles)
- [Microsoft Learn — Transfer Flexible Single Master Operations Roles](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/manage-fsmo-roles)
- [Microsoft Learn — View and Transfer FSMO Roles](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/view-transfer-fsmo-roles)
- [Microsoft Learn — Transfer or Seize Operation Master Roles](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/transfer-or-seize-operation-master-roles-in-ad-ds)
- [Microsoft Learn — Netdom Query](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netdom-query)
