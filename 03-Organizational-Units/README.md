# 03 - Organizational Units

## Overview

This section documents the Organizational Unit (OU) design implemented in the **VIREXON.LOCAL** Active Directory environment.

The OU hierarchy follows an enterprise-grade administrative model that separates users, computers, servers, security groups, and service accounts by department. This structure simplifies administration, supports delegated management, streamlines Group Policy deployment, and provides a scalable foundation for future infrastructure growth.

---

## OU Hierarchy

```text
VIREXON.LOCAL
│
├── Users
│   ├── IT
│   ├── HR
│   ├── Finance
│   ├── Sales
│   ├── Management
│   └── Marketing
│
├── Computers
│   ├── IT
│   ├── HR
│   ├── Finance
│   ├── Sales
│   ├── Management
│   └── Marketing
│
├── Servers
├── Groups
└── Service Accounts
```

---

## Organizational Unit Purpose

| Organizational Unit | Purpose |
|---------------------|---------|
| **Users/\*** | Stores departmental user accounts for administration, delegated management, and User Configuration Group Policy assignment. |
| **Computers/\*** | Stores departmental workstation computer objects, allowing Computer Configuration policies to be managed independently from user accounts. |
| **Servers** | Contains member servers isolated from workstation policies for dedicated server administration and security hardening. |
| **Groups** | Stores security groups used for permission management following the AGDLP model. |
| **Service Accounts** | Stores service accounts separately from standard users to improve security and simplify administration. |

---

## Design Principles

- Separate user accounts from computer objects.
- Use identical departmental structures for both **Users** and **Computers**.
- Isolate servers from workstation administration.
- Store security groups in a dedicated Organizational Unit.
- Separate service accounts from standard user accounts.
- Prepare the environment for scalable Group Policy deployment.
- Keep the OU hierarchy simple, consistent, and easy to expand.

---

## Naming Convention

- Department names use standard English naming:
  - IT
  - HR
  - Finance
  - Sales
  - Management
  - Marketing
- Top-level Organizational Units each have a single administrative purpose.
- The OU hierarchy is designed for long-term scalability, consistency, and maintainability.

---

## Benefits

- Improved administrative organization.
- Simplified Group Policy management.
- Clear separation of users, computers, servers, groups, and service accounts.
- Better scalability for future departments and infrastructure growth.
- Enterprise-ready Active Directory design aligned with Microsoft best practices.

---

## Screenshots

The *Screenshots* folder contains:

- 01-Root-Organizational-Unit.png
- 02-Top-Level-Organizational-Units.png
- 03-Users-Department-Structure.png
- 04-Computers-Department-Structure.png

---

## Status

Completed


