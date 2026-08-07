# Group Policy

## Overview

This section documents the implementation, testing, optimization, and final validation of Group Policy within the **VIREXON.LOCAL** Active Directory environment.

The objective was to implement practical Group Policy controls that simulate a structured corporate environment while maintaining appropriate separation between user-based and computer-based policies.

The Group Policy implementation followed this workflow:

**Create → Configure → Test → Review → Optimize → Validate**

---

## Group Policy Architecture

The Group Policy design is based on the Active Directory OU structure established for the VIREXON.LOCAL environment.

VIREXON.LOCAL
│
├── Corporate
│   │
│   ├── Users
│   │   ├── IT
│   │   ├── HR
│   │   ├── Finance
│   │   ├── Marketing
│   │   ├── Sales
│   │   └── Management
│   │
│   └── Computers
│       ├── IT
│       ├── HR
│       ├── Finance
│       ├── Marketing
│       ├── Sales
│       └── Management
│
├── Groups
└── Servers

---

# Implemented Group Policies

## 1. Corporate Desktop Wallpaper

**GPO:** `GPO-Corporate-Desktop-Wallpaper`

**Configuration:** User Configuration

**Final Linking Location:** `Corporate → Users`

The policy applies the corporate desktop wallpaper to users within the Corporate Users OU structure.

### Documentation

![GPO Corporate Desktop Wallpaper - Created](Screenshots/01-GPO-Corporate-Desktop-Wallpaper-Created.png)

![GPO Corporate Desktop Wallpaper - Linked](Screenshots/02-GPO-Corporate-Desktop-Wallpaper-Linked.png)

![GPO Corporate Desktop Wallpaper - Configured](Screenshots/03-GPO-Corporate-Desktop-Wallpaper-Configured.png)

![GPO Corporate Desktop Wallpaper - Applied](Screenshots/04-GPO-Corporate-Desktop-Wallpaper-Applied.png)

---

## 2. Prevent Control Panel and Settings

**GPO:** `GPO-Prevent-Control-Panel`

**Configuration:** User Configuration

**Final Target OUs:**

- HR
- Finance
- Marketing
- Sales

The policy prevents targeted departmental users from accessing Windows Control Panel and Settings.

The IT and Management OUs remain outside the current restriction scope.

### Documentation

![GPO Prevent Control Panel - Created](Screenshots/05-GPO-Prevent-Control-Panel-Created.png)

![GPO Prevent Control Panel - Linked](Screenshots/06-GPO-Prevent-Control-Panel-Linked.png)

![GPO Prevent Control Panel - Configured](Screenshots/07-GPO-Prevent-Control-Panel-Configured.png)

![GPO Prevent Control Panel - Applied](Screenshots/08-GPO-Prevent-Control-Panel-Applied.png)

---

## 3. Disable Command Prompt

**GPO:** `GPO-Disable-Command-Prompt`

**Configuration:** User Configuration

**Final Target OUs:**

- HR
- Finance
- Marketing
- Sales

The policy restricts access to Command Prompt for targeted departmental users.

The IT and Management OUs remain outside the current restriction scope.

### Documentation

![GPO Disable Command Prompt - Created](Screenshots/09-GPO-Disable-Command-Prompt-Created.png)

![GPO Disable Command Prompt - Linked](Screenshots/10-GPO-Disable-Command-Prompt-Linked.png)

![GPO Disable Command Prompt - Configured](Screenshots/11-GPO-Disable-Command-Prompt-Configured.png)

![GPO Disable Command Prompt - Applied](Screenshots/12-GPO-Disable-Command-Prompt-Applied.png)

---

## 4. Disable Registry Editor

**GPO:** `GPO-Disable-Registry-Editor`

**Configuration:** User Configuration

**Final Target OUs:**

- HR
- Finance
- Marketing
- Sales

The policy restricts access to Windows Registry Editor for targeted departmental users.

The IT and Management OUs remain outside the current restriction scope.

### Documentation

![GPO Disable Registry Editor - Created](Screenshots/13-GPO-Disable-Registry-Editor-Created.png)

![GPO Disable Registry Editor - Linked](Screenshots/14-GPO-Disable-Registry-Editor-Linked.png)

![GPO Disable Registry Editor - Configured](Screenshots/15-GPO-Disable-Registry-Editor-Configured.png)

![GPO Disable Registry Editor - Applied](Screenshots/16-GPO-Disable-Registry-Editor-Applied.png)

---

## 5. Remove Run Command

**GPO:** `GPO-Remove-Run-Command`

**Configuration:** User Configuration

**Final Target OUs:**

- HR
- Finance
- Marketing
- Sales

The policy removes access to the Windows Run command for targeted departmental users.

The IT and Management OUs remain outside the current restriction scope.

### Documentation

![GPO Remove Run Command - Created](Screenshots/17-GPO-Remove-Run-Command-Created.png)

![GPO Remove Run Command - Linked](Screenshots/18-GPO-Remove-Run-Command-Linked.png)

![GPO Remove Run Command - Configured](Screenshots/19-GPO-Remove-Run-Command-Configured.png)

![GPO Remove Run Command - Applied](Screenshots/20-GPO-Remove-Run-Command-Applied.png)

---

## 6. Disable Task Manager

**GPO:** `GPO-Disable-Task-Manager`

**Configuration:** User Configuration

**Final Target OUs:**

- HR
- Finance
- Marketing
- Sales

The policy prevents targeted departmental users from accessing Task Manager.

The IT and Management OUs remain outside the current restriction scope.

### Documentation

![GPO Disable Task Manager - Created](Screenshots/21-GPO-Disable-Task-Manager-Created.png)

![GPO Disable Task Manager - Linked](Screenshots/22-GPO-Disable-Task-Manager-Linked.png)

![GPO Disable Task Manager - Configured](Screenshots/23-GPO-Disable-Task-Manager-Configured.png)

![GPO Disable Task Manager - Applied](Screenshots/24-GPO-Disable-Task-Manager-Applied.png)

---

## 7. Disable USB Storage

**GPO:** `GPO-Disable-USB-Storage`

**Configuration:** Computer Configuration

**Final Linking Location:** `Corporate → Computers`

The policy restricts USB storage access on computers within the Corporate Computers OU structure.

### Documentation

![GPO Disable USB Storage - Created](Screenshots/25-GPO-Disable-USB-Storage-Created.png)

![GPO Disable USB Storage - Linked](Screenshots/26-GPO-Disable-USB-Storage-Linked.png)

![GPO Disable USB Storage - Configured](Screenshots/27-GPO-Disable-USB-Storage-Configured.png)

![GPO Disable USB Storage - Applied](Screenshots/28-GPO-Disable-USB-Storage-Applied.png)

---

## 8. Password Policy

**GPO:** `GPO-Password-Policy`

**Configuration:** Domain Security Policy

**Final Linking Location:** `VIREXON.LOCAL`

### Configuration

- Minimum password age: **30 days**
- Maximum password age: **90 days**
- Password history: **3 passwords**
- Password complexity: **Enabled**
- Minimum password length: **8 characters**

The policy was validated from the Windows 11 client by attempting to change a password that did not meet the configured password requirements.

### Documentation

![GPO Password Policy - Created](Screenshots/29-GPO-Password-Policy-Created.png)

![GPO Password Policy - Linked](Screenshots/30-GPO-Password-Policy-Linked.png)

![GPO Password Policy - Configured](Screenshots/31-GPO-Password-Policy-Configured.png)

![GPO Password Policy - Applied](Screenshots/32-GPO-Password-Policy-Applied.png)

![GPO Password Policy - Validation](Screenshots/33-GPO-Password-Policy-Validation.png)

---

## 9. Account Lockout Policy

**GPO:** `GPO-Account-Lockout-Policy`

**Configuration:** Domain Security Policy

**Final Linking Location:** `VIREXON.LOCAL`

### Configuration

- Account lockout threshold: **5 invalid logon attempts**
- Account lockout duration: **15 minutes**
- Reset account lockout counter after: **15 minutes**
- Allow Administrator account lockout: **Enabled**

The policy was validated using the Windows 11 client and the `net accounts` command.

### Documentation

![GPO Account Lockout Policy - Created](Screenshots/34-GPO-Account-Lockout-Policy-Created.png)

![GPO Account Lockout Policy - Linked](Screenshots/35-GPO-Account-Lockout-Policy-Linked.png)

![GPO Account Lockout Policy - Configured](Screenshots/36-GPO-Account-Lockout-Policy-Configured.png)

---

## 10. Interactive Logon Message

**GPO:** `GPO-Interactive-Logon-Message`

**Configuration:** Computer Configuration

**Final Linking Location:** `Corporate → Computers`

The policy displays an interactive logon message before authentication on computers within the targeted computer scope.

### Documentation

![GPO Interactive Logon Message - Created](Screenshots/37-GPO-Interactive-Logon-Message-Created.png)

![GPO Interactive Logon Message - Linked](Screenshots/38-GPO-Interactive-Logon-Message-Linked.png)

![GPO Interactive Logon Message - Configured](Screenshots/39-GPO-Interactive-Logon-Message-Configured.png)

![GPO Interactive Logon Message - Applied](Screenshots/40-GPO-Interactive-Logon-Message-Applied.png)

---

# Group Policy Optimization

After completing the initial implementation and testing of the ten Group Policy objects, the configuration was reviewed and reorganized according to policy type and intended scope.

The optimization focused on:

- Separating User Configuration policies from Computer Configuration policies.
- Moving domain security policies to the domain root.
- Limiting user restriction policies to the intended departmental OUs.
- Keeping the corporate wallpaper within the Corporate Users scope.
- Keeping computer-based policies within the Corporate Computers scope.
- Removing unnecessary use of **Enforced**.
- Validating the final linking structure.
- Validating representative user behavior after optimization.

---

## Final Domain-Level GPO Linking

The following policies are linked directly to the `VIREXON.LOCAL` domain root:

- `GPO-Password-Policy`
- `GPO-Account-Lockout-Policy`

These policies are domain-level security policies and were therefore retained at the domain root.

![Final Domain Root GPO Linking](Screenshots/41-GPO-Final-Domain-Root-Linking.png)

---

## Final Corporate OU Architecture

The final Group Policy structure uses the Corporate OU hierarchy to separate user-based and computer-based policy deployment.

User-based policies are scoped under:

`Corporate → Users`

Computer-based policies are scoped under:

`Corporate → Computers`

![Final Corporate OU Architecture](Screenshots/42-GPO-Final-Corporate-OU-Architecture.png)

---

## Final User GPO Scope

The final user-based GPO design is intentionally scoped according to the intended departmental requirements.

### Corporate Users

The following policy is linked to the Corporate Users OU:

- `GPO-Corporate-Desktop-Wallpaper`

### Departmental User OUs

The following user restriction policies are linked to:

- HR
- Finance
- Marketing
- Sales

Policies:

- `GPO-Prevent-Control-Panel`
- `GPO-Disable-Command-Prompt`
- `GPO-Disable-Task-Manager`
- `GPO-Disable-Registry-Editor`
- `GPO-Remove-Run-Command`

The IT and Management OUs remain outside the current restriction scope.

This provides the required distinction between standard departmental users and the IT/Management users without introducing Security Filtering at this stage.

![Final Users GPO Scope](Screenshots/43-GPO-Final-Users-GPO-Scope.png)

---

## Final Computer GPO Scope

The following computer-based policies are scoped under the Corporate Computers OU:

- `GPO-Interactive-Logon-Message`
- `GPO-Disable-USB-Storage`

This separates computer-based controls from the user-based restrictions.

![Final Computers GPO Scope](Screenshots/44-GPO-Final-Computers-GPO-Scope.png)

---

# Final Validation

## HR User Validation

A representative HR user was used to verify that the departmental user restrictions were successfully applied.

The validation confirmed that targeted restrictions, including Control Panel and Run, were enforced for the HR user.

![HR User Policy Validation](Screenshots/45-GPO-Validation-HR-Restrictions-Applied.png)

---

## IT User Validation

A representative IT user was used to verify that the departmental user restrictions were not applied outside the targeted departmental OUs.

The validation confirmed that Run and Control Panel remained accessible for the IT user.

![IT User Policy Validation](Screenshots/46-GPO-Validation-IT-Restrictions-Excluded.png)

---

# Security Filtering

Security Filtering and advanced group-based GPO targeting were intentionally not implemented during this phase.

The project will address these concepts later after they are covered in the corresponding Active Directory training modules.

This approach also avoids mixing the existing departmental security groups, such as `GG-HR`, `GG-Finance`, `GG-Marketing`, and `GG-Sales`, with GPO-specific targeting before the Group Scope, Group Nesting, Security Filtering, and File Server permission concepts are covered.

---

# Final Status

**Group Policy — Completed**

The Group Policy phase contains:

- **10 implemented GPOs**
- Defined policy scopes
- User-based and computer-based policy separation
- Domain-level security policies
- Department-specific user restrictions
- Tested policy behavior
- Final optimized linking structure
- Final HR and IT validation
- **46 documented screenshots**

The Group Policy implementation and optimization phase is complete.
