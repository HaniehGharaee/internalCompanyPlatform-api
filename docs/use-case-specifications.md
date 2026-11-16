# Use Case Specification — Internal Company Platform

## Document Information

| Field | Value |
|---|---|
| Project Name | Internal Company Platform |
| Document | Use Case Specification |
| Version | 1.0 |
| Status | V1 Requirements Baseline |
| Backend | NestJS |
| Frontend | React |
| Database | MongoDB |
| Object Storage | MinIO |

## 1. Purpose

This document describes how the actors interact with the Internal Company Platform to achieve their goals.

It complements `requirements.md` and `diagrams/use-case-diagram.md` by providing textual descriptions of the use cases, including preconditions, main flows, alternative flows, and postconditions.

## 2. Scope

This document covers the Version 1 (V1) use cases for:

- Company registration requests
- Company management
- System user administration
- Security and system settings
- Manager team management
- File upload, access, download, and assignment

Features outside the agreed V1 scope are not included in this specification.

## 3. Actors

### 3.1 Company Representative

A person who submits a company registration request to the platform.

The Company Representative does not need to log in to submit a registration request.

### 3.2 System Administrator

The actor responsible for system-wide administration, including company registration requests, company management, user administration, security logs, account unlocking, and system settings.

### 3.3 Manager

An authenticated user who manages team members and performs file-related operations within the scope of their authorized teams.

The system has one application role named `Manager`. Titles such as Company Manager, Finance Manager, Technical Manager, and Frontend Manager are descriptive titles, not separate application roles.

### 3.4 Employee

An authenticated user who can upload files and view and download files they are authorized to access.

### 3.5 External Systems

- **MongoDB:** Stores application data, including company, user, team, file metadata, and registration request records.
- **MinIO:** Stores uploaded file content.

These external systems support the platform's use cases and are not human actors.

## 4. Use Case Overview

### 4.1 Company Representative

- Submit Company Registration Request

### 4.2 System Administrator

**Company Registration Requests**
- View Requests
- Approve Request
- Reject Request

**Company Management**
- View Companies
- Edit Company
- Change Company Status
- Delete Company

**User Administration**
- View System Users
- Create User
- Edit User
- Delete User
- Manage User Roles
- Manage User Account Status

**Security & System Settings**
- View Security Logs
- Unlock User Account
- Manage System Settings

### 4.3 Manager

- Log in
- View Team Members
- Add Team Member
- Edit Team Member
- Remove Team Member
- Upload File
- View Accessible Files
- Download File
- Assign File to Employee

### 4.4 Employee

- Log in
- Upload File
- View Accessible Files
- Download File

## 5. Use Case Specifications

### UC-01: Submit Company Registration Request

**Primary Actor:** Company Representative

**Goal:** Submit a request to register a company on the platform.

**Preconditions:**
- The Company Representative can access the company registration form.

**Main Flow:**
1. The Company Representative opens the registration form.
2. The representative enters the required company information.
3. The representative enters the required information for the initial Manager.
4. The representative submits the request.
5. The system validates the submitted information.
6. The system records the registration request for administrative review.
7. The system indicates that the request has been submitted.

**Alternative Flows:**
- If required information is missing or invalid, the system identifies the validation errors and requests corrections.
- If the request cannot be submitted, the system informs the representative that submission was unsuccessful.

**Postconditions:**
- A registration request is recorded for review.
- The company is not activated merely because a request has been submitted.

### UC-02: View Requests

**Primary Actor:** System Administrator

**Goal:** View submitted company registration requests.

**Preconditions:**
- The System Administrator is authenticated and authorized to access registration requests.

**Main Flow:**
1. The administrator opens Company Registration Requests.
2. The system retrieves the available requests.
3. The system displays the request information and its current status.
4. The administrator selects a request if further review is needed.

**Alternative Flows:**
- If no requests are available, the system displays an empty state.

**Postconditions:**
- The administrator has viewed the available registration requests.
- No request status is changed by viewing the requests alone.

### UC-03: Approve Request

**Primary Actor:** System Administrator

**Goal:** Approve a company registration request and initiate company registration.

**Preconditions:**
- The administrator is authenticated and authorized.
- The selected request exists and is eligible for approval.

**Main Flow:**
1. The administrator selects a registration request.
2. The administrator reviews the submitted information.
3. The administrator approves the request.
4. The system records the approval.
5. The system creates the company record.
6. The system creates the initial Manager account using the information supplied in the request and the applicable account-creation rules.
7. The system records the resulting registration state.

**Alternative Flows:**
- If the request is no longer eligible for approval, the system prevents the invalid transition and informs the administrator.
- If the approval process fails, the system reports the failure and avoids leaving the registration in an incorrectly completed state.

**Postconditions:**
- The request is approved.
- The company and its initial Manager account are created as part of the approval process.

**Business Rule:** Company creation and initial Manager account creation are consequences of approving a registration request. They are not separate administrator use cases in the V1 use case overview.

### UC-04: Reject Request

**Primary Actor:** System Administrator

**Goal:** Reject a company registration request.

**Preconditions:**
- The administrator is authenticated and authorized.
- The selected request exists and is eligible for rejection.

**Main Flow:**
1. The administrator opens a registration request.
2. The administrator reviews the submitted information.
3. The administrator rejects the request.
4. The system records the rejection and updates the request status.
5. The system displays the result.

**Alternative Flows:**
- If the request is no longer eligible for rejection, the system prevents the invalid transition.

**Postconditions:**
- The request is recorded as rejected.
- No company or initial Manager account is created as a result of this rejected request.

### UC-05: View Companies

**Primary Actor:** System Administrator

**Goal:** View registered companies.

**Preconditions:**
- The administrator is authenticated and authorized to manage companies.

**Main Flow:**
1. The administrator opens Company Management.
2. The system retrieves the company records.
3. The system displays the available company information and statuses.
4. The administrator selects a company to view its details.

**Alternative Flows:**
- If no companies are registered, the system displays an empty state.

**Postconditions:**
- Company information is displayed.
- No company data is modified.

### UC-06: Edit Company

**Primary Actor:** System Administrator

**Goal:** Update a company's information.

**Preconditions:**
- The administrator is authenticated and authorized.
- The selected company exists.

**Main Flow:**
1. The administrator selects a company.
2. The system displays its editable information.
3. The administrator modifies the required fields.
4. The administrator submits the changes.
5. The system validates the changes.
6. The system saves the updated company information.
7. The system confirms the update.

**Alternative Flows:**
- If the submitted information is invalid, the system identifies the validation errors.
- If the company does not exist, the system reports that the operation cannot be completed.

**Postconditions:**
- The company's information is updated if validation and saving succeed.

### UC-07: Change Company Status

**Primary Actor:** System Administrator

**Goal:** Change a company's status.

**Preconditions:**
- The administrator is authenticated and authorized.
- The selected company exists.

**Main Flow:**
1. The administrator selects a company.
2. The system displays its current status.
3. The administrator selects the intended status.
4. The system validates the status change.
5. The system saves the new status.
6. The system displays the result.

**Alternative Flows:**
- If the requested status change is invalid, the system rejects the operation and informs the administrator.

**Postconditions:**
- The company has the new status if the operation succeeds.

**Note:** The exact company statuses and the effects of each status on user access are defined in the relevant requirements and detailed design.

### UC-08: Delete Company

**Primary Actor:** System Administrator

**Goal:** Delete a company and its associated platform data according to the system's deletion rules.

**Preconditions:**
- The administrator is authenticated and authorized.
- The selected company exists.

**Main Flow:**
1. The administrator selects a company.
2. The system presents the company information and deletion action.
3. The administrator confirms the deletion.
4. The system initiates the company deletion process.
5. The system deletes or schedules the deletion of the company's associated records and stored files according to the defined deletion strategy.
6. The system reports the outcome.

**Alternative Flows:**
- If the administrator cancels, no deletion is initiated.
- If the deletion process fails partially, the system handles or records the failure so that it can be recovered without affecting other companies' data.

**Postconditions:**
- The company and its associated data are deleted according to the system's deletion policy, or the failure is recorded for recovery.

**Business Rule:** Company deletion must not unintentionally delete or modify another company's data or files.

### UC-09: View System Users

**Primary Actor:** System Administrator

**Goal:** View user accounts across the platform.

**Preconditions:**
- The administrator is authenticated and authorized to administer system users.

**Main Flow:**
1. The administrator opens User Administration.
2. The system retrieves the user accounts.
3. The system displays the available user information, roles, account statuses, and associated company information where applicable.
4. The administrator selects a user to view further details.

**Alternative Flows:**
- If no user accounts are available, the system displays an empty state.

**Postconditions:**
- User information is displayed without modifying the accounts.

### UC-10: Create User

**Primary Actor:** System Administrator

**Goal:** Create a system user account through system-wide user administration.

**Preconditions:**
- The administrator is authenticated and authorized.
- The information required to create the account is available.

**Main Flow:**
1. The administrator opens User Administration.
2. The administrator selects Create User.
3. The administrator enters the required user information.
4. The administrator selects the applicable role and company association, where required.
5. The system validates the submitted information.
6. The system creates the account.
7. The system displays the result.

**Alternative Flows:**
- If required information is invalid, the system requests corrections.
- If the account conflicts with an existing identity, the system applies the account uniqueness and identity-handling rules.

**Postconditions:**
- A user account is created if the operation succeeds.

**Note:** This use case represents system-wide administration. It is distinct from a Manager adding a team member through the Manager panel.

### UC-11: Edit User

**Primary Actor:** System Administrator

**Goal:** Update a system user's information.

**Preconditions:**
- The administrator is authenticated and authorized.
- The selected user account exists.

**Main Flow:**
1. The administrator selects a user account.
2. The system displays the user's editable information.
3. The administrator updates the required fields.
4. The system validates the changes.
5. The system saves the updated information.
6. The system confirms the result.

**Alternative Flows:**
- If the submitted information is invalid, the system requests corrections.
- If the user account cannot be found, the system reports the issue.

**Postconditions:**
- The user's information is updated if the operation succeeds.

### UC-12: Delete User

**Primary Actor:** System Administrator

**Goal:** Delete a user account according to the system's account and data-retention rules.

**Preconditions:**
- The administrator is authenticated and authorized.
- The selected user account exists.

**Main Flow:**
1. The administrator selects a user account.
2. The system displays the account information.
3. The administrator initiates and confirms deletion.
4. The system processes the deletion according to the applicable rules.
5. The system reports the outcome.

**Alternative Flows:**
- If the administrator cancels, the account remains unchanged.
- If deletion is not permitted under the applicable rules, the system rejects the operation and explains the result.

**Postconditions:**
- The account is deleted if the operation succeeds.

**Note:** The treatment of related records and historical audit information is subject to the system's retention policy.

### UC-13: Manage User Roles

**Primary Actor:** System Administrator

**Goal:** Manage the application roles assigned to system users.

**Preconditions:**
- The administrator is authenticated and authorized.
- The selected user account exists.

**Main Flow:**
1. The administrator selects a user account.
2. The system displays the user's current role.
3. The administrator selects the intended role.
4. The system validates the role assignment.
5. The system saves the role change.
6. The system confirms the result.

**Alternative Flows:**
- If the selected role is invalid or the change is not permitted, the system rejects the operation.

**Postconditions:**
- The user's role is updated if the operation succeeds.

**Business Rule:** `Manager` is one application role. Titles such as Finance Manager and Technical Manager are descriptive job titles, not separate application roles.

### UC-14: Manage User Account Status

**Primary Actor:** System Administrator

**Goal:** Change a user's account status.

**Preconditions:**
- The administrator is authenticated and authorized.
- The selected user account exists.

**Main Flow:**
1. The administrator selects a user account.
2. The system displays its current status.
3. The administrator selects the intended status.
4. The system validates the status change.
5. The system saves the new status.
6. The system confirms the result.

**Alternative Flows:**
- If the requested status change is invalid, the system rejects the operation.

**Postconditions:**
- The account has the new status if the operation succeeds.

**Note:** Administrative account status changes are distinct from unlocking an account following repeated failed login attempts.

### UC-15: View Security Logs

**Primary Actor:** System Administrator

**Goal:** Inspect security-related events recorded by the platform.

**Preconditions:**
- The administrator is authenticated and authorized to access security logs.

**Main Flow:**
1. The administrator opens Security Logs.
2. The system retrieves the available security log entries.
3. The system displays the recorded events and relevant information.
4. The administrator reviews the entries.

**Alternative Flows:**
- If no entries are available, the system displays an empty state.

**Postconditions:**
- The administrator has viewed the available security logs.
- Viewing logs does not alter the recorded events.

### UC-16: Unlock User Account

**Primary Actor:** System Administrator

**Goal:** Restore login eligibility for an account that has been locked.

**Preconditions:**
- The administrator is authenticated and authorized.
- The selected account exists and is locked.

**Main Flow:**
1. The administrator selects the locked account.
2. The system displays its current account state.
3. The administrator initiates the unlock operation.
4. The system validates that the account can be unlocked.
5. The system removes the applicable lock.
6. The system confirms the result.

**Alternative Flows:**
- If the account is not locked or cannot be unlocked, the system informs the administrator.

**Postconditions:**
- The account is unlocked if the operation succeeds.

**Note:** Unlocking an account does not necessarily change other account statuses, such as an administrator-imposed inactive status.

### UC-17: Manage System Settings

**Primary Actor:** System Administrator

**Goal:** Manage the system settings exposed to authorized administrators.

**Preconditions:**
- The administrator is authenticated and authorized to manage system settings.

**Main Flow:**
1. The administrator opens System Settings.
2. The system displays the available settings.
3. The administrator changes one or more supported settings.
4. The system validates the submitted values.
5. The system saves the valid changes.
6. The system confirms the result.

**Alternative Flows:**
- If a setting value is invalid, the system identifies the validation error.
- If the changes cannot be saved, the system reports the failure.

**Postconditions:**
- The supported settings are updated if the operation succeeds.

**Note:** The exact configurable settings will be specified during detailed design.

### UC-18: Log in

**Primary Actors:** Manager, Employee

**Goal:** Authenticate a user and provide access to the appropriate part of the platform.

**Preconditions:**
- The user has an account.
- The account is eligible to log in.

**Main Flow:**
1. The user opens the login page.
2. The user enters their credentials.
3. The system validates the credentials and account state.
4. The system establishes an authenticated session if authentication succeeds.
5. The system provides access according to the user's role and permissions.

**Alternative Flows:**
- If the credentials are invalid, the system rejects the login attempt.
- If the account is locked or otherwise ineligible to log in, the system denies access and displays an appropriate message.
- Repeated failed attempts are handled according to the authentication security rules.

**Postconditions:**
- The user is authenticated and granted appropriate access, or access is denied.

### UC-19: View Team Members

**Primary Actor:** Manager

**Goal:** View members of teams the Manager is authorized to manage.

**Preconditions:**
- The Manager is authenticated.
- The Manager has access to at least one relevant team or team membership record.

**Main Flow:**
1. The Manager opens team management.
2. The system retrieves the team members within the Manager's authorized scope.
3. The system displays the available member information.
4. The Manager selects a member to view further information, where supported.

**Alternative Flows:**
- If no team members are available, the system displays an empty state.
- If the Manager attempts to access a team outside their authorized scope, the system denies access.

**Postconditions:**
- The authorized team member information is displayed.

### UC-20: Add Team Member

**Primary Actor:** Manager

**Goal:** Add a user to a team the Manager is authorized to manage.

**Preconditions:**
- The Manager is authenticated.
- The Manager is authorized to manage the relevant team.

**Main Flow:**
1. The Manager opens team management.
2. The Manager selects the appropriate team.
3. The Manager enters the required information for the person to add.
4. The system validates the information and checks the relevant identity and membership rules.
5. The system creates or associates the user account as appropriate under the account-creation rules.
6. The system adds the user to the selected team.
7. The system confirms the result.

**Alternative Flows:**
- If the Manager lacks permission for the selected team, the system rejects the operation.
- If the submitted information is invalid, the system requests corrections.
- If the person already has an account, the system applies the defined existing-account rules rather than blindly creating a duplicate account.
- If the user is already a member of the team, the system avoids creating a duplicate membership.

**Postconditions:**
- The user is added to the selected team if the operation succeeds.

**Business Rules:**
- Managers can add team members through the Manager panel.
- A Manager's authority is limited to the teams they are authorized to manage.
- A user may belong to multiple teams, and a team may have multiple Managers.
- Adding a team member is distinct from system-wide user administration by the System Administrator.

### UC-21: Edit Team Member

**Primary Actor:** Manager

**Goal:** Update permitted information for a team member within the Manager's authorized scope.

**Preconditions:**
- The Manager is authenticated.
- The member belongs to a team the Manager is authorized to manage.

**Main Flow:**
1. The Manager opens the relevant team.
2. The Manager selects a team member.
3. The system displays the information the Manager is permitted to edit.
4. The Manager updates the required fields.
5. The system validates the changes and the Manager's permissions.
6. The system saves the changes.
7. The system confirms the result.

**Alternative Flows:**
- If the Manager lacks permission, the system rejects the operation.
- If the submitted information is invalid, the system requests corrections.

**Postconditions:**
- The permitted team member information is updated if the operation succeeds.

**Note:** The exact fields editable by a Manager will be specified during detailed design.

### UC-22: Remove Team Member

**Primary Actor:** Manager

**Goal:** Remove a user from a team the Manager is authorized to manage.

**Preconditions:**
- The Manager is authenticated.
- The member belongs to a team the Manager is authorized to manage.

**Main Flow:**
1. The Manager opens the relevant team.
2. The Manager selects a team member.
3. The Manager initiates the removal.
4. The system validates the Manager's permissions.
5. The system removes the user's membership in the selected team.
6. The system confirms the result.

**Alternative Flows:**
- If the Manager lacks permission, the system rejects the operation.
- If the user is not a member of the selected team, the system reports that the operation cannot be completed.

**Postconditions:**
- The user's membership in the selected team is removed.

**Business Rules:**
- Removing a user from one team does not automatically remove them from other teams.
- Removing team membership does not automatically delete the user account.
- Existing file assignments are handled according to the defined file-access rules; removal from a team does not automatically revoke previously granted file access.

### UC-23: Upload File

**Primary Actors:** Manager, Employee

**Goal:** Upload a file to the platform.

**Preconditions:**
- The user is authenticated.
- The user is authorized to upload files.

**Main Flow:**
1. The user opens the file upload interface.
2. The user selects a file and provides any required metadata.
3. The system validates the file and submitted information.
4. The system stores the file content in MinIO.
5. The system records the file metadata and relevant ownership or company association in MongoDB.
6. The system confirms the successful upload.

**Alternative Flows:**
- If the file or metadata fails validation, the system rejects the upload and explains the issue.
- If file storage or metadata persistence fails, the system handles the failure to avoid leaving an incorrectly completed upload.
- If the user is not authorized to upload, the system denies the operation.

**Postconditions:**
- The file is stored and its metadata is recorded if the operation succeeds.

### UC-24: View Accessible Files

**Primary Actors:** Manager, Employee

**Goal:** View files the user is authorized to access.

**Preconditions:**
- The user is authenticated.

**Main Flow:**
1. The user opens the file list.
2. The system retrieves files accessible to the user according to ownership and assignment permissions.
3. The system displays the available file information.
4. The user selects a file to view its supported details.

**Alternative Flows:**
- If no files are accessible, the system displays an empty state.
- If a user attempts to access a file outside their permissions, the system denies access.

**Postconditions:**
- The user sees only files they are authorized to access.

**Business Rules:**
- Users can access their own uploaded files.
- Assignment of a file to an employee grants the access defined by the system's file-access rules.
- Team membership alone does not automatically grant access to every file uploaded by other team members.

### UC-25: Download File

**Primary Actors:** Manager, Employee

**Goal:** Download a file the user is authorized to access.

**Preconditions:**
- The user is authenticated.
- The requested file exists.
- The user has permission to download the file.

**Main Flow:**
1. The user selects a file from the accessible file list.
2. The user initiates the download.
3. The system verifies the user's access permissions.
4. The system retrieves the file content from MinIO.
5. The system provides the file for download.

**Alternative Flows:**
- If the user is not authorized, the system denies the download.
- If the file is unavailable in storage, the system reports the failure.

**Postconditions:**
- The file is delivered to the user if the operation succeeds.

### UC-26: Assign File to Employee

**Primary Actor:** Manager

**Goal:** Grant an eligible employee access to a file.

**Preconditions:**
- The Manager is authenticated.
- The Manager is authorized to manage the relevant team.
- The file exists and the Manager has permission to assign it.
- The target employee is eligible for assignment under the team's access rules.

**Main Flow:**
1. The Manager opens the file list.
2. The Manager selects a file.
3. The Manager selects an eligible employee.
4. The system validates the Manager's permissions, the employee's eligibility, and the file's assignment rules.
5. The system records the assignment.
6. The system confirms that the file has been assigned.

**Alternative Flows:**
- If the Manager is not authorized to assign the file to the selected employee, the system rejects the operation.
- If the file or employee does not exist or is ineligible, the system reports the issue.
- If the assignment already exists, the system avoids creating a duplicate assignment.

**Postconditions:**
- The assignment is recorded if the operation succeeds.
- The employee receives the view and download access granted by the assignment.

**Business Rules:**
- A Manager cannot assign files to employees outside the Manager's authorized team scope.
- Assignment grants access to the assigned file; it does not grant general access to all files belonging to the Manager or team.
- Removing a user from a team does not automatically revoke existing assignments.

## 6. Cross-Cutting Business Rules

The following rules apply across multiple use cases.

- **BR-01 — Company Isolation:** A company's data and files must remain isolated from those of other companies.
- **BR-02 — Registration Approval:** A company and its initial Manager account are created as part of approving a company registration request.
- **BR-03 — Single Manager Role:** The application has one `Manager` role. Manager titles are descriptive and do not create separate roles.
- **BR-04 — Manager Scope:** A Manager may manage only teams and resources within their authorized scope.
- **BR-05 — Team Membership:** A user may belong to multiple teams, and a team may have multiple Managers.
- **BR-06 — User and Membership Separation:** Removing a user from a team does not automatically delete the user account or remove memberships in other teams.
- **BR-07 — File Ownership:** Users can access their own uploaded files, subject to applicable access and account rules.
- **BR-08 — File Assignment:** Assigning a file to an eligible employee grants the defined view and download permissions.
- **BR-09 — Team Membership Is Not File Ownership:** Team membership alone does not grant access to all files uploaded by team members.
- **BR-10 — Company Deletion:** Deleting a company must not affect another company's data or stored files.
- **BR-11 — Authentication and Authorization:** Protected operations require authentication and appropriate authorization.
- **BR-12 — Security Events:** Relevant security events must be handled according to the system's security logging requirements.

## 7. Traceability to Requirements

Each use case should map to one or more functional requirements in `requirements.md`.

The following table provides the conceptual mapping. Exact requirement identifiers should match those in the finalized requirements document.

| Use Case Group | Related Requirement Area |
|---|---|
| Submit Company Registration Request | Company registration |
| View, Approve, Reject Requests | Registration request administration |
| View, Edit, Change Status, Delete Company | Company management |
| View, Create, Edit, Delete System Users | System user administration |
| Manage User Roles and Account Status | Roles and account administration |
| View Security Logs, Unlock User Account | Security and account protection |
| Manage System Settings | System configuration |
| Log in | Authentication |
| View, Add, Edit, Remove Team Members | Team management |
| Upload, View, Download Files | File management and access control |
| Assign File to Employee | File assignment and authorization |

## 8. Items for Detailed Design

The following details do not need to be fully resolved at the use-case overview stage. They should be specified before implementing the relevant functionality.

- Exact fields required for company registration and Manager creation.
- The complete set of company and user account statuses and their effects.
- Which user fields Managers can edit.
- Existing-account handling when adding a team member, including users associated with another company.
- File validation rules, size limits, and supported file types.
- The technical strategy for safely handling company deletion across MongoDB and MinIO.
- The initial credential policy and password-change requirements.
- The exact configurable system settings.
- The precise audit and security event retention policy.

These items should be resolved in the relevant design documents and implementation stages without changing the agreed V1 use-case scope unnecessarily.

## 9. Related Documents

- `requirements.md` — Functional and non-functional requirements.
- `diagrams/use-case-diagram.png` — Visual overview of actors and use cases.
- `diagrams/use-case-diagram.drawio` — Editable source of the use case diagram.
- `architecture.md` — High-level system architecture.
- `data-model.md` — Data entities and relationships.
- `api-design.md` — API endpoints and request/response contracts.
