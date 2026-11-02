# Data Model — Internal Company Platform

**Version:** 1.0  
**Status:** Initial Data Model Baseline  
**Scope:** Version 1 (V1)

---

## 1. Purpose

This document describes the data model of the Internal Company Platform.

It identifies the main entities, their attributes, relationships, and business constraints required to support the system's V1 functionality.

The data model serves as a reference for database design and implementation. It also helps maintain consistency between system requirements, use cases, architecture, and database structures.

The system uses MongoDB as its primary database. The entities and relationships described in this document represent the logical data model and do not necessarily dictate the exact MongoDB document structure.

## 2. Entities

The V1 data model consists of the following eight entities:

1. User
2. Company
3. Team
4. TeamMembership
5. File
6. FileAssignment
7. CompanyRegistrationRequest
8. SecurityLog

### 2.1 User

Represents a user account within the platform.

A user belongs to one company. System Administrators are global users and are not associated with a company.

**Attributes:**

| Attribute | Description |
|---|---|
| `id` | Unique identifier of the user |
| `firstName` | User's first name |
| `lastName` | User's last name |
| `nationalId` | User's national identification number |
| `phoneNumber` | User's phone number |
| `email` | User's email address |
| `username` | Username used for authentication |
| `passwordHash` | Hashed password |
| `role` | User's platform role |
| `status` | User account status |
| `companyId` | Reference to the associated company; optional for System Administrators |
| `createdAt` | Account creation date |
| `updatedAt` | Date of the last account update |

**Notes:**

- `role` identifies the user's platform role, such as `Manager`, `Employee`, or `System Administrator`.
- Manager is a single application role. Organizational responsibilities or team labels do not create additional platform roles.
- A user can belong to multiple teams within their company through the `TeamMembership` entity.

### 2.2 Company

Represents an organization registered on the platform.

**Attributes:**

| Attribute | Description |
|---|---|
| `id` | Unique identifier of the company |
| `name` | Company name |
| `phone` | Company contact number |
| `email` | Company email address |
| `status` | Company status |
| `createdAt` | Company creation date |
| `updatedAt` | Date of the last company update |

**Notes:**

- A company can have multiple users and teams.
- Each company has its own data scope.
- Company data must remain isolated from other companies.

### 2.3 Team

Represents a team within a company.

**Attributes:**

| Attribute | Description |
|---|---|
| `id` | Unique identifier of the team |
| `name` | Team name |
| `description` | Team description |
| `companyId` | Reference to the company that owns the team |
| `createdAt` | Team creation date |
| `updatedAt` | Date of the last team update |

**Notes:**

- Each team belongs to exactly one company.
- A company can have multiple teams.
- A team can have multiple members through `TeamMembership`.
- Team names identify teams for users but are not used as relationship identifiers.

### 2.4 TeamMembership

Represents the membership relationship between a user and a team.

This entity manages the many-to-many relationship between `User` and `Team`.

**Attributes:**

| Attribute | Description |
|---|---|
| `id` | Unique identifier of the membership |
| `userId` | Reference to the user |
| `teamId` | Reference to the team |
| `joinedAt` | Date the user joined the team |
| `status` | Membership status |
| `addedBy` | Reference to the user who added the member |

**Notes:**

- A user can belong to multiple teams.
- A team can contain multiple users.
- A user can only join teams belonging to their own company.
- Adding a team member is performed by a Manager in V1.
- Removing a user from one team does not automatically remove them from other teams.
- Team membership is managed independently of the user's platform account.

### 2.5 File

Represents file metadata stored in the platform.

The actual file content is stored in MinIO, while its metadata and storage reference are maintained in the database.

**Attributes:**

| Attribute | Description |
|---|---|
| `id` | Unique identifier of the file |
| `originalName` | Original filename provided by the user |
| `storageKey` | Object key used to locate the file in MinIO |
| `mimeType` | MIME type of the file |
| `size` | File size |
| `uploadedBy` | Reference to the user who uploaded the file |
| `companyId` | Reference to the company that owns the file |
| `createdAt` | File upload date |
| `updatedAt` | Date of the last metadata update |

**Notes:**

- File content is stored in MinIO, not directly in MongoDB.
- `originalName` is used to identify and display the filename to users.
- `storageKey` identifies the corresponding object in MinIO.
- `mimeType` describes the file's content type.
- Each file belongs to one company.
- File ownership is associated with the company, not directly with a team.
- File access is controlled through the platform's authorization rules and file assignments.

### 2.6 FileAssignment

Represents the assignment of a file to a user.

It records which user assigned a file and which user received it.

**Attributes:**

| Attribute | Description |
|---|---|
| `id` | Unique identifier of the assignment |
| `fileId` | Reference to the assigned file |
| `assignedTo` | Reference to the user receiving the file |
| `assignedBy` | Reference to the user who assigned the file |
| `assignedAt` | Date and time of the assignment |

**Notes:**

- In V1, only users with the `Manager` role can assign files.
- A Manager can assign files to eligible users within their authorized scope.
- An assignment grants the recipient permission to view and download the file through the platform.
- A user may receive multiple file assignments.
- A user may assign multiple files.
- The assignment record does not imply that previously downloaded file copies can be removed from the recipient's device.
- Team-based file assignment by other team members is outside the scope of V1 and may be considered in future versions.

### 2.7 CompanyRegistrationRequest

Represents a request submitted by a company representative to register a company on the platform.

The request contains company information and the personal information required to create the initial Manager account after approval.

**Attributes:**

| Attribute | Description |
|---|---|
| `id` | Unique identifier of the registration request |
| `companyName` | Name of the company |
| `companyPhone` | Company contact number |
| `companyEmail` | Company email address |
| `managerFirstName` | First name of the initial Manager |
| `managerLastName` | Last name of the initial Manager |
| `managerNationalId` | National identification number of the initial Manager |
| `managerPhone` | Phone number of the initial Manager |
| `status` | Registration request status |
| `submittedAt` | Date and time the request was submitted |
| `reviewedAt` | Date and time the request was reviewed |
| `reviewedBy` | Reference to the System Administrator who reviewed the request |
| `rejectionReason` | Reason for rejection, if applicable |
| `createdCompanyId` | Reference to the company created after approval |

**Notes:**

- A company representative can submit a registration request without logging in.
- Submitting a request does not immediately create an active company.
- The System Administrator can approve or reject the request.
- Upon approval, the company and its initial Manager account are created.
- `reviewedBy` identifies the specific System Administrator who performed the review.
- `createdCompanyId` remains empty until the request is approved and the company is created.
- The registration request is independent of the `User` entity because the representative does not need an existing platform account to submit it.

### 2.8 SecurityLog

Represents a security-related event recorded by the platform.

**Attributes:**

| Attribute | Description |
|---|---|
| `id` | Unique identifier of the security event |
| `userId` | Reference to the associated user, if available |
| `companyId` | Reference to the associated company, if applicable |
| `eventType` | Type of security event |
| `description` | Description of the event |
| `ipAddress` | IP address associated with the event |
| `userAgent` | Client or browser information |
| `createdAt` | Date and time the event was recorded |

**Notes:**

- Security events can be associated with a user and/or a company.
- `userId` may be empty when the event cannot be associated with an existing user, such as a login attempt using an unknown username.
- `companyId` may be empty for platform-wide events.
- Security logs are used for monitoring and auditing security-related activities.

---

## 3. Entity Relationships

The following relationships define how the main entities are connected.

| Entity A | Relationship | Entity B | Cardinality |
|---|---|---|---|
| Company | employs | User | One-to-Many |
| Company | contains | Team | One-to-Many |
| Company | owns | File | One-to-Many |
| User | has | TeamMembership | One-to-Many |
| Team | has | TeamMembership | One-to-Many |
| User | uploads | File | One-to-Many |
| File | has | FileAssignment | One-to-Many |
| User | receives | FileAssignment | One-to-Many |
| User | assigns | FileAssignment | One-to-Many |
| User | reviews | CompanyRegistrationRequest | One-to-Many, optional on the request side |
| CompanyRegistrationRequest | creates | Company | One-to-One, optional until approval |
| User | is associated with | SecurityLog | One-to-Many, optional on the log side |
| Company | is associated with | SecurityLog | One-to-Many, optional on the log side |

### Relationship Details

**User and Team**

The relationship between `User` and `Team` is many-to-many and is managed through `TeamMembership`.

**User and FileAssignment**

User has two distinct relationships with `FileAssignment`:

- `assignedTo`: identifies the recipient.
- `assignedBy`: identifies the user who performed the assignment.

These are separate relationships even though both reference the same entity.

**CompanyRegistrationRequest and Company**

A registration request can create a company after approval. A rejected or pending request does not have an associated created company.

**SecurityLog**

A security event may be associated with a user, a company, both, or neither, depending on the event.

---

## 4. Business Constraints

The following constraints must be enforced by the application:

1. Each regular user belongs to only one company.
2. System Administrators are global users and do not belong to a company.
3. Each team belongs to exactly one company.
4. Users can only join teams belonging to their own company.
5. A user cannot have duplicate active membership in the same team.
6. A Manager can manage only teams within their authorized scope.
7. In V1, only Managers can assign files to users.
8. File assignments must respect company isolation and the Manager's authorization scope.
9. Users can access files they uploaded or files assigned to them, subject to authorization rules.
10. Removing a user's team membership does not automatically revoke existing file assignments.
11. A company registration request must be approved before the company and its initial Manager account are created.
12. Company data and file access must remain isolated between companies.

---

## 5. Data Storage Considerations

The platform uses MongoDB for structured application data and MinIO for file content storage.

The logical entities described in this document may be implemented as separate MongoDB collections or, where appropriate, through embedded documents and references.

The use of identifiers such as `userId`, `companyId`, and `teamId` represents logical relationships between entities.

MongoDB does not automatically enforce relational foreign-key constraints. Therefore, reference integrity and business constraints must be handled by the application.

### File Storage

- File metadata is stored in MongoDB.
- Actual file content is stored in MinIO.
- `storageKey` connects a database file record to its corresponding MinIO object.
- File upload and deletion workflows must account for consistency between database records and MinIO objects.

---

## 6. Scope and Future Considerations

The current data model describes V1 functionality.

The following capabilities are not included in the current V1 data model:

- Team-based file assignment by regular team members.
- Tasks and project management.
- Notifications.
- Video calling and real-time communication.
- Advanced search and filtering.
- Detailed system settings management.

Some future capabilities may require additional entities, attributes, relationships, or changes to authorization rules.

The current model should be extended when those features are formally included in the project scope.

---

## 7. Related Documents

- [Requirements Specification](requirements.md)
- [Use Case Specification](use-cases.md)
- [Architecture](architecture.md)
- [Entity Relationship Diagram](diagrams/erd-v1.png)

