# Requirements Specification — Company Document Management Platform

**Version:** 1.0  
**Status:** V1 Requirements Baseline  
**Backend:** NestJS  
**Frontend:** React  
**Database:** MongoDB  
**Object Storage:** MinIO

---

## 1. Introduction

### 1.1 Purpose

This document defines the functional and non-functional requirements for Version 1 (V1) of the Company Document Management Platform.

It serves as the baseline for system design, implementation, and testing.

### 1.2 Project Overview

The platform provides a centralized environment for companies to manage users, teams, and files.

Companies can register through a request-and-approval process. Once a request is approved by a System Administrator, the company and its initial manager account are created or activated.

Managers can manage team membership and assign files within their authorized scope. Employees can upload files and access files they are authorized to view and download.

The platform enforces company-level data isolation and stores physical file content separately from file metadata.

---

## 2. Scope

### 2.1 In Scope

V1 includes:

- Authentication and account security.
- Company registration requests and administrative approval.
- Company and user administration.
- Manager and employee management.
- Manager organizational titles.
- Team creation and membership management.
- File upload, viewing, downloading, and assignment.
- Company-level data isolation.
- Security event logging and account management.
- Company deletion, including associated database records and physical files.

### 2.2 Out of Scope

The following features are planned for future versions:

- Task and project management.
- Deadlines and progress tracking.
- Weekly and monthly reports.
- Advanced search and notifications.
- Separate media management.
- Real-time communication and video calls.
- Redis- and WebSocket-based features.
- Advanced analytics.

---

## 3. Actors and Roles

### 3.1 Company Representative

A person who submits a company registration request without logging in.

Responsibilities:

- Provide company information.
- Provide the required information for the proposed initial manager.
- Submit the request for administrative review.

The representative is not automatically an application user.

### 3.2 System Administrator

A system-wide administrator responsible for:

- Reviewing, approving, and rejecting company registration requests.
- Managing companies and user accounts.
- Managing account statuses and application roles.
- Reviewing security logs.
- Unlocking blocked accounts.
- Managing system settings.

### 3.3 Manager

`Manager` is the single application role used by all managers.

A manager may have an organizational title displayed in the frontend, such as:

- Company Manager
- Technical Manager
- Finance Manager
- Human Resources Manager

These titles are display labels, not separate application roles.

The initial manager is the person specified in the approved company registration request. The Company Manager may add other managers to the company. Managers can manage teams and members within their authorized organizational scope.

### 3.4 Employee

Employees can:

- Log in and change their passwords.
- Upload files.
- View files they uploaded.
- View files assigned to them.
- Download files they are authorized to access.

Team membership alone does not grant access to other members' files.

---

## 4. Functional Requirements

### 4.1 Authentication and Account Security

**FR-01:** The system shall authenticate registered users using a username and password.

**FR-02:** During the initial V1 implementation, a user's mobile number shall be used as the username and national identification number as the initial password.

**FR-03:** Passwords shall never be stored in plaintext. The system shall use an appropriate password-hashing algorithm.

**FR-04:** Users shall be able to change their passwords after logging in.

**FR-05:** The system shall enforce account-status restrictions and protect against repeated failed login attempts.

**FR-06:** The System Administrator shall be able to unlock blocked accounts.

**FR-07:** The backend shall enforce authentication and authorization for protected resources.

### 4.2 Company Registration and Administration

**FR-08:** A Company Representative shall be able to submit a company registration request without authentication.

**FR-09:** The registration process shall validate submitted information and apply appropriate abuse-prevention controls.

**FR-10:** The System Administrator shall be able to review, approve, or reject registration requests.

**FR-11:** Approval shall create the company and create or activate the initial manager account. Submission alone shall not create an active company.

**FR-12:** The System Administrator shall be able to view, create, edit, change the status of, and delete companies through authorized operations.

**FR-13:** Deleting a company shall remove its associated database records, dependent records, and physical files stored in MinIO without affecting other companies.

### 4.3 User and Manager Management

**FR-14:** Authorized managers shall be able to add users using the required identity information, including mobile number and national identification number.

**FR-15:** If an account already exists for the recognized identity, the system shall reuse the existing account instead of creating a duplicate.

**FR-16:** If an account does not exist, the system shall create it according to the V1 initial-credential policy.

**FR-17:** The Company Manager shall be able to add other managers without assigning them to a team during account creation.

**FR-18:** Authorized managers shall be able to add employees within their permitted organizational scope.

**FR-19:** The system shall support a display title for managers, separate from the `Manager` application role.

**FR-20:** The System Administrator shall be able to manage user accounts and account statuses within the system-wide administrative scope.

### 4.4 Team Management

**FR-21:** An authorized manager shall be able to create and manage teams within the manager's permitted organizational scope.

**FR-22:** A team may have multiple managers, and a manager may be responsible for multiple teams.

**FR-23:** A user may belong to multiple teams.

**FR-24:** Authorized managers shall be able to add users to and remove users from teams within their scope.

**FR-25:** Removing a user from one team shall not automatically remove the user's membership in other teams.

**FR-26:** Removing a user from a team shall prevent new file assignments through that team unless the user becomes a member of that team again.

**FR-27:** A manager shall not manage teams or members outside the manager's authorized scope.

### 4.5 File Management

**FR-28:** Authorized users shall be able to upload files within their permitted company scope.

**FR-29:** Physical file content shall be stored in MinIO, while file metadata and storage references shall be stored in MongoDB.

**FR-30:** The system shall associate each file with its company and uploader.

**FR-31:** Users shall be able to view files they uploaded and files assigned to them, subject to applicable security restrictions.

**FR-32:** An authorized manager shall be able to assign a file to an eligible employee within the manager's permitted scope.

**FR-33:** File assignment shall grant the recipient permission to view and download the assigned file.

**FR-34:** Team membership alone shall not grant access to files uploaded by other team members.

**FR-35:** The backend shall verify authorization before providing file content or allowing a download.

**FR-36:** Removing a user from a team shall not automatically delete previously assigned files or revoke access solely because the membership ended.

### 4.6 Security and Data Integrity

**FR-37:** The system shall enforce company-level data isolation for users, teams, files, and other company-owned resources.

**FR-38:** The system shall record relevant security events, including failed login attempts, account lockouts, and important administrative actions.

**FR-39:** The System Administrator shall be able to review security logs.

**FR-40:** Sensitive authentication data shall not be exposed in logs or unauthorized API responses.

**FR-41:** The company-deletion workflow shall detect and handle partial failures between MongoDB and MinIO, with retry or recovery support.

---

## 5. Business Rules

**BR-01 — Registration Approval:** A company must be approved by the System Administrator before becoming active through the registration workflow.

**BR-02 — Initial Manager:** The person specified in the approved registration request becomes the initial manager of the company.

**BR-03 — Single Manager Role:** All managers use the `Manager` application role. Organizational titles such as Company Manager and Technical Manager are display labels, not separate roles.

**BR-04 — Organizational Scope:** A manager's authority depends on authorized organizational responsibilities, not merely on the manager's display title.

**BR-05 — Team Relationships:** Managers and users may belong to multiple teams. A team may have multiple managers.

**BR-06 — Independent Membership:** Removing a user from one team shall not affect membership in other teams.

**BR-07 — Assignment Eligibility:** New file assignments must respect the manager's authority and the user's active membership in the relevant team or other explicitly authorized scope.

**BR-08 — File Access:** Users may access files they uploaded or files assigned to them, subject to applicable security restrictions. Shared team membership alone does not grant access.

**BR-09 — Existing Assignments:** Ending team membership does not automatically delete previously assigned files or revoke access solely because membership ended.

**BR-10 — Company Isolation:** Company-owned resources must remain isolated from other companies. All relevant access checks must be enforced by the backend.

**BR-11 — Company Deletion:** Company deletion is intended to be permanent in V1 and must include dependent database records and associated MinIO objects. Partial failures must be recoverable, and unrelated company data must remain unaffected.

**BR-12 — Initial Credentials:** V1 initially uses the national identification number as the initial password. Replacing it with a randomly generated temporary password and requiring a password change on first login is a planned security improvement.

---

## 6. Non-Functional Requirements

### 6.1 Security

- **NFR-01:** Protected backend endpoints shall enforce authentication and authorization.
- **NFR-02:** Company isolation shall be enforced on the backend, not only in the frontend.
- **NFR-03:** Passwords shall be stored using a suitable password-hashing algorithm.
- **NFR-04:** File access shall be authorized before file content is provided.
- **NFR-05:** Public registration endpoints shall implement appropriate validation and abuse prevention.
- **NFR-06:** Sensitive information shall not be exposed in logs or unauthorized responses.

### 6.2 Maintainability

- **NFR-07:** The backend shall use a modular architecture that separates controllers, business logic, and persistence concerns.
- **NFR-08:** Business rules shall not be implemented exclusively in controllers or frontend components.
- **NFR-09:** Roles, organizational titles, team membership, and file permissions shall remain conceptually separate.

### 6.3 Reliability and Testability

- **NFR-10:** The system shall maintain consistency between file metadata and physical file storage as far as practicable.
- **NFR-11:** Partial failures during company deletion shall be detectable and recoverable.
- **NFR-12:** Automated tests shall cover critical authentication, authorization, team membership, file assignment, and company-isolation rules.

### 6.4 Performance

- **NFR-13:** Physical file content shall be stored in object storage rather than embedded directly in MongoDB documents.
- **NFR-14:** List operations shall be designed to support pagination as datasets grow.

Specific performance targets will be defined when expected usage and deployment requirements are known.

---

## 7. Acceptance Criteria

V1 shall be considered functionally ready when the following criteria are satisfied:

- [ ] Company Representatives can submit registration requests without logging in.
- [ ] The System Administrator can review, approve, and reject requests.
- [ ] Approval creates the company and its initial manager account.
- [ ] Authentication, password hashing, account-status checks, and failed-login protection work as specified.
- [ ] The Company Manager can add other managers without assigning them to a team during account creation.
- [ ] Authorized managers can manage team membership within their permitted scope.
- [ ] A manager can be responsible for multiple teams, and a team can have multiple managers.
- [ ] A user can belong to multiple teams.
- [ ] Removing a user from one team does not remove membership in other teams.
- [ ] Users can upload files and access files they uploaded or that were assigned to them.
- [ ] File assignment grants the recipient permission to view and download the assigned file.
- [ ] Team membership alone does not grant access to other members' files.
- [ ] Removing a user from a team prevents new assignments through that team while preserving eligibility through other teams where membership remains active.
- [ ] Physical files are stored in MinIO and file metadata is stored in MongoDB.
- [ ] Backend authorization prevents unauthorized file access and cross-company data access.
- [ ] Company deletion removes its database records and associated MinIO objects without affecting unrelated companies.
- [ ] Automated tests cover critical security, authorization, team-membership, and data-integrity rules.

