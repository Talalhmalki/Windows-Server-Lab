# 04 - Users and Groups

## Overview

This phase focuses on building the enterprise identity structure inside Active Directory. User accounts, security groups, and computer objects were created and organized according to departmental roles and enterprise administration best practices.

---

# Objectives

- Create enterprise user accounts.
- Configure user account properties.
- Organize users by department.
- Create Global Security Groups.
- Assign users to the appropriate groups.
- Join a Windows 11 client to the Active Directory domain.
- Organize computer objects inside the correct Organizational Unit.
- Verify successful domain membership.

---

# User Management

Enterprise user accounts were created following a standardized naming convention. During creation, account settings and user attributes were configured to prepare the environment for centralized identity management.

### Screenshot 01

*New User Wizard*

![01-New-User-Wizard](Screenshots/01-New-User-Wizard.png)

---

Password policies were configured during account creation to ensure every user account followed the required security configuration.

### Screenshot 02

*User Password Configuration*

![02-User-Password-Configuration](Screenshots/02-User-Password-Configuration.png)

---

Additional user information such as department and job title was configured to improve administration and future Group Policy targeting.

### Screenshot 03

*User Properties*

![03-User-Properties-General](Screenshots/03-User-Properties-General.png)

---

After completing user creation, all enterprise users were organized inside their corresponding Organizational Units.

### Screenshot 04

*Users Created and Organized*

![04-Users-Created-and-Organized](Screenshots/04-Users-Created-and-Organized.png)

---

# Security Groups

Department-based Global Security Groups were created following Microsoft's AGDLP administration model.

### Screenshot 05

*New Security Group Wizard*

![05-New-Security-Group-Wizard](Screenshots/05-New-Security-Group-Wizard.png)

---

All required departmental security groups were successfully created.

### Screenshot 06

*Security Groups Created*

![06-Security-Groups-Created](Screenshots/06-Security-Groups-Created.png)

---

Before assigning users, group membership was verified to confirm that the newly created groups contained no members.

### Screenshot 07

*Group Membership Before Adding Users*

![07-Group-Membership-Before-Adding-Users](Screenshots/07-Group-Membership-Before-Adding-Users.png)

---

Users were then assigned to the appropriate departmental security groups based on their organizational roles.

### Screenshot 08

*Group Membership Configured*

![08-Group-Membership-Configured](Screenshots/08-Group-Membership-Configured.png)

---

# Computer Management

A Windows 11 Enterprise client was integrated into the Active Directory environment.

The workstation was prepared, joined to the domain, and verified before being organized inside the enterprise OU structure.

### Screenshot 09

*Computer Information*

![09-Computer-Information](Screenshots/09-Computer-Information.png)

---

After joining the domain, the computer object was moved from the default *Computers* container into the dedicated *IT Computers OU*.

### Screenshot 10

*Computer Moved to IT OU*

![10-Computer-Moved-To-IT-OU](Screenshots/10-Computer-Moved-To-IT-OU.png)

---

Finally, domain membership was verified from the client workstation to confirm successful integration with Active Directory.

### Screenshot 11

*Computer Domain Membership*

![11-Computer-Domain-Membership](Screenshots/11-Computer-Domain-Membership.png)

---

# Best Practices Applied

- Standardized naming convention
- Department-based Organizational Units
- Department-based Security Groups
- Enterprise user account structure
- Computer object organization
- Active Directory logical hierarchy
- Preparation for Group Policy deployment

---

# Result

The enterprise Active Directory environment now contains properly organized users, security groups, and managed computer objects. This structure provides the required foundation for centralized authentication, authorization, Group Policy deployment, and future enterprise infrastructure services.
