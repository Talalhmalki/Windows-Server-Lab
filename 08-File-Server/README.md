# 08 - File Server and Access Control

## Purpose

This phase implements departmental file services for VIREXON and records the complete permission-design lifecycle: storage and sharing, an initial over-permission issue, the corrected AGDLP-based model, NTFS and share authorization, Access-Based Enumeration (ABE), effective-access testing, SMB monitoring, DNS-based access, and Group Policy drive mapping.

## Verified environment

| Component | Configuration |
| --- | --- |
| File server | `PC26.virexon.local` |
| Server address | `192.168.1.2` |
| Server operating system | Windows Server 2025 Standard Evaluation |
| Data volume | `F:`, 60 GB |
| Department root | `F:\CompanyData\Departments` |
| Share | `\\PC26\Departments` |
| Client | `PC-IT-01` running Windows 11 |
| Mapped drive | `S:`, label `Company Department` |
| Departments | Finance, HR, IT, Management, Marketing, Sales |

## Storage and share design

The six department folders are stored beneath `F:\CompanyData\Departments` and published through the `Departments` SMB share. The fully qualified share path visible in Server Manager is `\\PC26.virexon.local\Departments`; the shorter `\\PC26\Departments` path resolves to the same share and is used by the drive-map preference.

## Permission-model evolution

### Historical first implementation

The early model nested an entire departmental group, such as `GG-Management-Users`, into a single Modify-oriented Domain Local group. The initial HR ACL and client tests proved department isolation, but the design granted every member of a department the same Modify capability.

Those screenshots are retained as implementation history. They are not presented as the final least-privilege model.

### Corrected directory structure

Three group categories were separated beneath `VIREXON\Groups`:

| Category | Function |
| --- | --- |
| Department groups | General departmental identity groups such as `GG-IT-Users` |
| Access-role groups | Global Reader and Editor groups |
| Resource-permission groups | Domain Local Read and Modify groups bound to NTFS |

The general `GG-<Department>-Users` groups remain useful as department identities, but the final folder ACLs do not grant them direct access. Resource access flows through the Reader or Editor role groups.

### Final AGDLP chain

```text
User account
    → GG-<Department>-Readers or GG-<Department>-Editors
    → DL-<Department>-Read or DL-<Department>-Modify
    → NTFS Read & execute or Modify
```

| Department | Reader role | Read permission | Editor role | Modify permission |
| --- | --- | --- | --- | --- |
| Finance | `GG-Finance-Readers` | `DL-Finance-Read` | `GG-Finance-Editors` | `DL-Finance-Modify` |
| HR | `GG-HR-Readers` | `DL-HR-Read` | `GG-HR-Editors` | `DL-HR-Modify` |
| IT | `GG-IT-Readers` | `DL-IT-Read` | `GG-IT-Editors` | `DL-IT-Modify` |
| Management | `GG-Management-Readers` | `DL-Management-Read` | `GG-Management-Editors` | `DL-Management-Modify` |
| Marketing | `GG-Marketing-Readers` | `DL-Marketing-Read` | `GG-Marketing-Editors` | `DL-Marketing-Modify` |
| Sales | `GG-Sales-Readers` | `DL-Sales-Read` | `GG-Sales-Editors` | `DL-Sales-Modify` |

The captures confirm that the access-role groups are Global and the resource-permission groups are Domain Local.

## Final NTFS model

Inheritance is disabled on each departmental folder. The final explicit ACL pattern is:

| Principal | Permission | Applies to |
| --- | --- | --- |
| `VIREXON\Administrators` | Full Control | This folder, subfolders, and files |
| `SYSTEM` | Full Control | This folder, subfolders, and files |
| `CREATOR OWNER` | Full Control | Subfolders and files only |
| `DL-<Department>-Read` | Read & execute | This folder, subfolders, and files |
| `DL-<Department>-Modify` | Modify | This folder, subfolders, and files |

PowerShell verification shows the department-specific Read and Modify entries across all six folders with no inherited department ACEs.

## Share permissions and ABE

The share layer grants `VIREXON\Domain Users` Full Control. This is deliberately broad at the SMB layer; the restrictive authorization boundary is the NTFS ACL on each department folder.

ABE is enabled on the `Departments` share. It improves usability by hiding folders a user cannot access, but it does not replace NTFS security. The client evidence shows the IT user seeing the IT folder while unauthorized department folders are hidden.

## Functional validation

### Department isolation

- An IT user was denied access to the HR folder.
- The same test context could access and modify the IT folder under the historical model.
- After redesign, Reader and Editor behavior was tested separately.

### Reader behavior

The IT Reader account `Ahmed Ali (A.Ali)` could traverse, list, and read the IT folder. Creating files or folders, writing attributes, deleting, changing permissions, taking ownership, and obtaining Full Control were denied.

### Editor behavior

The IT Editor account `Abdulaziz Alharbi (A.Alharbi)` could create and write data and had `Delete` permission. The effective-access capture also shows that `Delete subfolders and files`, Full Control, Change permissions, and Take ownership were not granted. This is correctly described as Modify-level access, not Full Control.

### Operational checks

- The SMB console showed an active session to the `Departments` share.
- The share was opened by DNS name rather than only by IP address.
- The final Reader and Editor results were calculated with Windows Effective Access as well as tested from the client.

## Group Policy drive mapping

The `Map Network Drive` GPO contains a user-side Drive Maps preference with:

| Setting | Value |
| --- | --- |
| Action | Update |
| Location | `\\PC26\Departments` |
| Label | `Company Department` |
| Drive letter | `S:` |
| Reconnect | No |

Client evidence shows the GPO in the applied policy list, a successful `gpupdate`, and the `Company Department (S:)` network location.

## Troubleshooting record

| Issue | Correction |
| --- | --- |
| Department membership implied Modify access | Added separate Reader and Editor Global groups. |
| Resource groups required the correct scope | Recreated the resource layer as Domain Local groups. |
| Drive mapping initially referenced the wrong path type | Used the UNC path `\\PC26\Departments`. |

These corrections preserve the implementation history while making the final architecture unambiguous.

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [File Server Role](Screenshots/01-File-Server-Role-Verification.png) | File Server role service is installed. |
| 02 | [Data Volume](Screenshots/02-File-Server-Data-Volume.png) | Dedicated 60 GB `F:` data volume. |
| 03 | [Department Folders](Screenshots/03-Department-Folder-Structure.png) | Six department folders under `F:\CompanyData\Departments`. |
| 04 | [Initial AGDLP Nesting](Screenshots/04-AGDLP-Group-Nesting.png) | Historical department-to-Modify nesting. |
| 05 | [Initial HR NTFS ACL](Screenshots/05-HR-NTFS-Permissions.png) | Historical HR permission assignment. |
| 06 | [Departments SMB Share](Screenshots/06-Departments-SMB-Share.png) | The local folder is published as `Departments`. |
| 07 | [Unauthorized Access Denied](Screenshots/07-Unauthorized-Department-Access-Denied.png) | IT user denied access to HR. |
| 08 | [Historical IT Modify Test](Screenshots/08-Authorized-IT-Modify-Access.png) | IT modification worked before role separation. |
| 09 | [AD Group Architecture](Screenshots/09-AD-Group-Architecture.png) | Group-category OUs. |
| 10 | [Access-Role Groups](Screenshots/10-Access-Role-Groups.png) | Reader and Editor Global groups. |
| 11 | [Resource-Permission Groups](Screenshots/11-Resource-Permission-Groups.png) | Read and Modify Domain Local groups. |
| 12 | [Department Global Groups](Screenshots/12-Department-Global-Groups.png) | General `GG-<Department>-Users` groups. |
| 13 | [IT Editors Membership](Screenshots/13-IT-Editors-Group-Membership.png) | IT Editor role membership. |
| 14 | [IT Readers Membership](Screenshots/14-IT-Readers-Group-Membership.png) | IT Reader role membership. |
| 15 | [Final AGDLP Nesting](Screenshots/15-AGDLP-Group-Membership-Verification.png) | Role groups nested into their matching resource groups. |
| 16 | [IT NTFS Permissions](Screenshots/16-IT-NTFS-Permissions.png) | Explicit final ACL and disabled inheritance. |
| 17 | [All Department ACLs](Screenshots/17-All-Departments-NTFS-Permissions-Verification.png) | Read and Modify ACEs across all departments. |
| 18 | [Reader Write Denied](Screenshots/18-IT-Reader-Access-Denied-Verification.png) | Reader cannot create content. |
| 19 | [Editor Modify Test](Screenshots/19-IT-Editor-Modify-Verification.png) | Editor can create content. |
| 20 | [ABE Enabled](Screenshots/20-ABE-Enabled-on-Departments-Share.png) | ABE setting on the share. |
| 21 | [ABE Client Result](Screenshots/21-ABE-Client-Verification.png) | Unauthorized departments are hidden. |
| 22 | [Share Permissions](Screenshots/22-Departments-Share-Permissions.png) | Domain Users Full Control at the share layer. |
| 23 | [Reader Effective Access](Screenshots/23-IT-Reader-Effective-Access.png) | Exact allowed and denied Reader operations. |
| 24 | [Editor Effective Access](Screenshots/24-IT-Editor-Effective-Access.png) | Exact allowed and denied Editor operations. |
| 25 | [Active SMB Session](Screenshots/25-SMB-Active-Session-Verification.png) | Live access to the share is visible server-side. |
| 26 | [DNS-Name Share Access](Screenshots/26-DNS-Name-Share-Access-Verification.png) | `\\PC26\Departments` resolves and opens. |
| 27 | [Drive Map Configuration](Screenshots/27-GPO-Drive-Map-Configuration.png) | `S:` preference settings and UNC path. |
| 28 | [Drive Map Client Result](Screenshots/28-GPO-Drive-Map-Client-Verification.png) | Applied GPO and visible mapped drive. |

## Outcome

The final implementation provides a centralized departmental share with role-based AGDLP authorization, explicit NTFS permissions, broad-but-controlled share permissions, ABE, client and effective-access validation, operational SMB evidence, DNS-based access, and centralized drive mapping. The historical over-permission state remains documented without being confused with the final design.
