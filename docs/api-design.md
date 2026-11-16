# API Design — Internal Company Platform

**Version:** 0.1  
**Status:** Initial API Design  
**Scope:** Version 1 (V1)  
**Backend:** NestJS  
**Frontend:** React  
**Database:** MongoDB  
**Object Storage:** MinIO

---

## 1. Purpose

This document defines the initial REST API design for the Internal Company Platform.

It describes the API endpoints, HTTP methods, access requirements, request parameters, and expected responses needed to support the V1 use cases.

The API design is based on the following documents:

- `requirements.md`
- `use-cases.md`
- `data-model.md`

This document serves as the initial API contract between the frontend and backend and will be refined during implementation.

## 2. API Overview

The platform exposes a RESTful API.

### 2.1 Base URL

```text
/api/v1
```

### 2.2 Content Types

- Request format: `application/json`
- Response format: `application/json`
- File upload: `multipart/form-data`
- File download: Binary file response

### 2.3 Authentication

Protected endpoints require an authenticated user.

The API must identify the authenticated user's role and enforce authorization rules before executing protected operations.

### 2.4 User Roles

| Role | Description |
|---|---|
| System Administrator | Global system administration |
| Manager | Manages authorized teams and performs file operations |
| Employee | Uploads and accesses authorized files |

The `Manager` role is a single application role. Titles such as Finance Manager or Technical Manager are not separate application roles.

## 3. API Conventions

### 3.1 HTTP Methods

| Method | Purpose |
|---|---|
| GET | Retrieve resources |
| POST | Create a resource or perform an operation |
| PATCH | Partially update a resource |
| DELETE | Delete a resource |

### 3.2 Common Response Structure

Successful responses should use a consistent JSON structure where applicable.

Example:

```json
{
  "success": true,
  "data": {
    "id": "resource-id"
  },
  "message": "Operation completed successfully."
}
```

The exact response structure may be refined during implementation.

### 3.3 Common Error Responses

The API should return appropriate HTTP status codes.

| Status Code | Description |
|---|---|
| 400 | Bad Request — Invalid request data |
| 401 | Unauthorized — Authentication required or invalid |
| 403 | Forbidden — Insufficient permissions |
| 404 | Not Found — Resource does not exist |
| 409 | Conflict — Duplicate or conflicting operation |
| 422 | Unprocessable Entity — Business rule validation failure |
| 500 | Internal Server Error |

Example:

```json
{
  "success": false,
  "message": "You are not authorized to perform this operation.",
  "statusCode": 403
}
```

## 4. Authentication APIs

Authentication endpoints support user login and access to protected platform resources.

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/auth/login` | Authenticate a user | Public |

### POST `/auth/login`

**Request Body:**

```json
{
  "username": "user123",
  "password": "password"
}
```

**Successful Response:**

```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user-id",
      "firstName": "John",
      "lastName": "Doe",
      "role": "Manager",
      "companyId": "company-id"
    },
    "accessToken": "token"
  }
}
```

**Possible Errors:**

- Invalid credentials.
- Account is locked.
- Account is inactive or otherwise ineligible to log in.

Repeated failed login attempts must be handled according to the authentication security rules.

The exact token strategy, expiration policy, and refresh-token mechanism are to be defined during detailed design.

## 5. Company Registration APIs

These endpoints support public company registration requests and administrative review.

### 5.1 Public Registration

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/registration-requests` | Submit a company registration request | Public |

### POST `/registration-requests`

**Request Body:**

```json
{
  "companyName": "Example Company",
  "companyPhone": "02112345678",
  "companyEmail": "info@example.com",
  "managerFirstName": "John",
  "managerLastName": "Doe",
  "managerNationalId": "1234567890",
  "managerPhone": "09123456789"
}
```

**Successful Response:**

```json
{
  "success": true,
  "data": {
    "id": "request-id",
    "status": "Pending",
    "submittedAt": "2026-11-15T10:00:00Z"
  },
  "message": "Registration request submitted successfully."
}
```

Submitting a request does not create or activate a company.

### 5.2 Registration Request Administration

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/admin/registration-requests` | View registration requests | System Administrator |
| GET | `/admin/registration-requests/:id` | View a specific request | System Administrator |
| PATCH | `/admin/registration-requests/:id/approve` | Approve a request | System Administrator |
| PATCH | `/admin/registration-requests/:id/reject` | Reject a request | System Administrator |

**Approval Behavior:**

When a request is approved:

1. The request status is updated.
2. The company record is created.
3. The initial Manager account is created.
4. The created company reference is stored in the registration request.

These operations must be handled consistently so that an unsuccessful approval does not leave an incorrectly completed registration.

**Rejection Request Body:**

```json
{
  "rejectionReason": "Required information is incomplete."
}
```

The exact validation and initial account credential rules are to be defined during detailed design.

## 6. Company Management APIs

These endpoints allow the System Administrator to manage registered companies.

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/admin/companies` | View all companies | System Administrator |
| GET | `/admin/companies/:id` | View company details | System Administrator |
| PATCH | `/admin/companies/:id` | Edit company information | System Administrator |
| PATCH | `/admin/companies/:id/status` | Change company status | System Administrator |
| DELETE | `/admin/companies/:id` | Delete a company | System Administrator |

### PATCH `/admin/companies/:id`

**Request Body:**

```json
{
  "name": "Updated Company Name",
  "phone": "02112345678",
  "email": "contact@example.com"
}
```

### PATCH `/admin/companies/:id/status`

**Request Body:**

```json
{
  "status": "Inactive"
}
```

The available company statuses and their effects on user access must be defined during detailed design.

Company deletion must respect the data-retention and storage-deletion rules and must not affect other companies.

## 7. User Administration APIs

These endpoints provide system-wide user administration.

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/admin/users` | View system users | System Administrator |
| GET | `/admin/users/:id` | View user details | System Administrator |
| POST | `/admin/users` | Create a user account | System Administrator |
| PATCH | `/admin/users/:id` | Edit user information | System Administrator |
| DELETE | `/admin/users/:id` | Delete a user account | System Administrator |
| PATCH | `/admin/users/:id/role` | Manage user role | System Administrator |
| PATCH | `/admin/users/:id/status` | Manage account status | System Administrator |

### POST `/admin/users`

**Request Body:**

```json
{
  "firstName": "John",
  "lastName": "Doe",
  "nationalId": "1234567890",
  "phoneNumber": "09123456789",
  "email": "john@example.com",
  "username": "john123",
  "password": "password",
  "role": "Employee",
  "companyId": "company-id"
}
```

The request fields and initial credential rules are subject to detailed design.

### PATCH `/admin/users/:id/role`

**Request Body:**

```json
{
  "role": "Manager"
}
```

### PATCH `/admin/users/:id/status`

**Request Body:**

```json
{
  "status": "Active"
}
```

The System Administrator has global user administration access.

This API group is separate from Manager team-member management.

## 8. Team Management APIs

These endpoints allow Managers to manage members of their authorized teams.

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/teams/:teamId/members` | View team members | Manager |
| POST | `/teams/:teamId/members` | Add a team member | Manager |
| PATCH | `/teams/:teamId/members/:userId` | Edit permitted team-member information | Manager |
| DELETE | `/teams/:teamId/members/:userId` | Remove a team member | Manager |

### GET `/teams/:teamId/members`

Returns the members of a team within the Manager's authorized scope.

### POST `/teams/:teamId/members`

**Request Body:**

```json
{
  "firstName": "Jane",
  "lastName": "Smith",
  "nationalId": "1234567890",
  "phoneNumber": "09123456789",
  "email": "jane@example.com",
  "username": "jane123"
}
```

The system must determine whether the person needs a new user account or an existing account should be associated with the team.

The exact existing-account handling rules are to be defined during detailed design.

A successful operation creates or associates the user as appropriate and creates a `TeamMembership` record.

### PATCH `/teams/:teamId/members/:userId`

Updates only the information that a Manager is permitted to edit.

The exact editable fields are to be defined during detailed design.

### DELETE `/teams/:teamId/members/:userId`

Removes the user's membership from the selected team.

This operation must not automatically delete the user account or remove memberships in other teams.

Existing file assignments are not automatically revoked by team membership removal.

**Authorization Rules:**

- A Manager can manage only authorized teams.
- Users can belong to multiple teams.
- A team can have multiple Managers.
- A user can only join teams belonging to their own company.
- Duplicate active memberships must be prevented.

## 9. File Management APIs

These endpoints support file upload, file access, download, and assignment.

### 9.1 File Upload

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/files` | Upload a file | Manager, Employee |

### POST `/files`

**Content-Type:** `multipart/form-data`

**Request Fields:**

| Field | Type | Description |
|---|---|---|
| file | File | File content |
| Additional metadata | To Be Defined | Optional file information |

**Successful Response:**

```json
{
  "success": true,
  "data": {
    "id": "file-id",
    "originalName": "document.pdf",
    "mimeType": "application/pdf",
    "size": 102400,
    "uploadedBy": "user-id",
    "companyId": "company-id",
    "createdAt": "2026-11-15T10:00:00Z"
  }
}
```

The actual file content is stored in MinIO.

File metadata and the corresponding `storageKey` are stored in MongoDB.

The `storageKey` should not be exposed as a public storage location.

### 9.2 View Accessible Files

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/files` | View accessible files | Manager, Employee |
| GET | `/files/:id` | View accessible file details | Manager, Employee |

The system returns only files the authenticated user is authorized to access.

Access is granted according to the following rules:

- Users can access files they uploaded themselves.
- Users can access files assigned to them.
- Team membership alone does not grant access to all files uploaded by other team members.

### 9.3 Download File

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/files/:id/download` | Download an accessible file | Manager, Employee |

Before providing the file, the system must verify the user's permission.

The file content is retrieved from MinIO and returned as a download response.

### 9.4 Assign File to Employee

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/files/:id/assignments` | Assign a file to an employee | Manager |

**Request Body:**

```json
{
  "assignedTo": "employee-user-id"
}
```

**Successful Response:**

```json
{
  "success": true,
  "data": {
    "id": "assignment-id",
    "fileId": "file-id",
    "assignedTo": "employee-user-id",
    "assignedBy": "manager-user-id",
    "assignedAt": "2026-11-15T10:00:00Z"
  }
}
```

The assignment creates a `FileAssignment` record.

**Business Rules:**

- Only Managers can assign files in V1.
- The Manager can assign files only within their authorized scope.
- The target employee must be eligible under the team's access rules.
- Duplicate assignments must be prevented.
- Assignment grants view and download access.
- Removing a user from a team does not automatically revoke existing assignments.

## 10. Security and System APIs

These endpoints support security monitoring and administrative operations.

### 10.1 Security Logs

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/admin/security-logs` | View security logs | System Administrator |
| GET | `/admin/security-logs/:id` | View a specific security log | System Administrator |

The returned information may include:

- User ID, if available
- Company ID, if applicable
- Event type
- Description
- IP address
- User agent
- Event creation date

### 10.2 Unlock User Account

| Method | Endpoint | Description | Access |
|---|---|---|---|
| PATCH | `/admin/users/:id/unlock` | Unlock a locked user account | System Administrator |

This operation removes the applicable login lock.

It must not automatically change other account statuses, such as an administrator-imposed inactive status.

### 10.3 System Settings

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/admin/settings` | View supported system settings | System Administrator |
| PATCH | `/admin/settings` | Update supported system settings | System Administrator |

The exact configurable settings and their data structure are to be defined during detailed design.

## 11. Authorization Rules

Authorization must be enforced on the backend.

The frontend must not be treated as the authority for access control.

### 11.1 Role-Based Access

| Resource / Operation | System Administrator | Manager | Employee |
|---|---|---|---|
| Registration requests | Yes | No | No |
| Company management | Yes | No | No |
| System user administration | Yes | No | No |
| Security logs | Yes | No | No |
| System settings | Yes | No | No |
| Team member management | No | Authorized teams only | No |
| Upload files | No | Yes | Yes |
| View accessible files | No | Yes | Yes |
| Download accessible files | No | Yes | Yes |
| Assign files | No | Authorized scope only | No |

### 11.2 Company Isolation

All company-related operations must enforce company-level data isolation.

A user must not access another company's users, teams, or files.

System Administrator operations are global and must follow their administrative authorization rules.

### 11.3 File Access

File access must be checked for each request.

A user can access a file if:

1. The user uploaded the file; or
2. The file has been assigned to the user.

Team membership alone does not grant access to every file belonging to team members.

## 12. Data Model Mapping

The API operations interact with the following logical entities:

| Entity | Related API Groups |
|---|---|
| User | Authentication, User Administration, Team Management |
| Company | Company Registration, Company Management |
| Team | Team Management |
| TeamMembership | Team Management |
| File | File Management |
| FileAssignment | File Assignment |
| CompanyRegistrationRequest | Company Registration |
| SecurityLog | Security and System |

MongoDB stores application data, while MinIO stores actual file content.

## 13. API Design Considerations

The following details remain open and should be resolved before implementing their respective features:

- Authentication token strategy and session management.
- Exact request and response DTO fields.
- Company and user status values and their effects.
- Initial Manager account credential creation.
- Existing-account handling when adding a team member.
- Fields a Manager is allowed to edit.
- File size limits and supported MIME types.
- File upload and deletion consistency between MongoDB and MinIO.
- Company deletion strategy.
- Security log event types and retention policy.
- Configurable system settings.
- Pagination and advanced filtering strategy, if included in the V1 implementation scope.

These decisions should be made during detailed design without unnecessarily expanding the agreed V1 use cases.

## 14. Out of Scope

The following features are not part of this V1 API design:

- Project and task management.
- Notifications.
- Video calls and real-time communication.
- Advanced search and filtering.
- Team-wide automatic file access.
- File assignment by regular employees or other non-Manager users.

These capabilities may require additional endpoints and data model changes in future versions.

## 15. Related Documents

- `requirements.md` — Functional and non-functional requirements.
- `use-cases.md` — Use Case specifications.
- `diagrams/use-case-diagram.png` — Use Case diagram.
- `data-model.md` — Data entities and relationships.
- `architecture.md` — System architecture.

---

**Document Status:** Initial API Design — V1

This document is intended to evolve as the system's detailed design and backend implementation progress.