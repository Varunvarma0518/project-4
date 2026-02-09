Secure Healthcare Management System (SHMS). In a HIPAA-regulated environment, the software must satisfy the Security Rule, which focuses on Administrative, Physical, and Technical safeguards.

1. Architectural Philosophy: Zero Trust
The system is built on the Zero Trust Model. Traditionally, security was like a castle (strong perimeter, weak interior). SHMS assumes the network is compromised, meaning:

Verify Explicitly: Every request must be authenticated and authorized.

Least Privilege: Users only have access to the data necessary for their role.

Assume Breach: Every interaction is logged and monitored for anomalies.

2. Technical Safeguards (HIPAA Compliance)
A. Identity and Access Management (IAM)
The system utilizes OAuth2 and OpenID Connect (OIDC) via Spring Security.

JWT (JSON Web Tokens): Used for stateless authentication. Each token contains claims (user roles, patient IDs) signed by a private key.

RBAC (Role-Based Access Control): Permissions are mapped to roles:

ROLE_PATIENT: Access to personal records only.

ROLE_DOCTOR: Access to assigned patient records and medical history.

ROLE_ADMIN: Access to system logs and user management, but not necessarily clinical data (Separation of Duties).

B. Multi-Factor Authentication (MFA)
To comply with HIPAA’s requirement for "person or entity authentication," we implement TOTP (Time-based One-Time Password). This adds a physical layer (the user's device) to the knowledge layer (password).

3. Data Protection Layer
A. Encryption in Transit
All communication between the client (Browser/Mobile) and the server is encrypted using TLS 1.3. This prevents "Man-in-the-Middle" (MITM) attacks.

Algorithm: AES-256-GCM.

Mechanism: Forced HTTPS and Secure Cookies.

B. Encryption at Rest
HIPAA requires that data stored on disks is unreadable if stolen.

Transparent Data Encryption (TDE): The database layer encrypts the physical files.

Application-Level Encryption: Sensitive fields (SSN, HIV status, Genetic data) are encrypted before being sent to the database. Even a database administrator with "SELECT" access cannot read the plain text without the application’s master key.

4. Integrity and Auditability
A. Audit Logging
HIPAA §164.312(b) requires "Audit Controls." Our system implements an automated trail:

Who accessed the data? (User ID)

When did they access it? (Timestamp)

What was the action? (Read, Update, Delete)

Source: (IP Address/Device ID)

B. Performance & Availability
A system that is offline is not "secure" for a patient in an emergency.
5. Security Testing Theory
We employ a Shift-Left Security approach, where security is tested during development, not just at the end.
Test Type,Objective,Tooling
SAST,Static analysis of Java code for vulnerabilities.,SonarQube
DAST,Dynamic testing of the running API.,OWASP ZAP
SCA,Checking libraries for known CVEs.,Dependency-Check
Unit Security,Testing @PreAuthorize logic.,JUnit / MockMvc

6. Summary of Compliance Mapping

HIPAA Requirement,SHMS Implementation
Access Control,OAuth2 + RBAC + JWT
Audit Controls,Spring AOP + Audit Log Table
Integrity,SHA-256 Hashing
Transmission Security,TLS 1.3
Automatic Logoff,JWT Expiration + Redis Session Management
<img width="250" height="200" alt="image" src="https://github.com/user-attachments/assets/b8a48963-c55a-4336-9f3e-514ec54d8867" />

Key Components of this Architecture:
Identity & Access Management (IAM):

Users & Roles: Implements the Many-to-Many relationship between users and roles to support RBAC (Role-Based Access Control).

Permissions: Granular permissions linked to roles for method-level security.

Medical Records (PHI Core):

Patients: Contains the sensitive personal information that requires application-level encryption (SSN, Phone, Address).

Encounters/Medical Records: Linked to both Patient and Doctor, forming the core of Protected Health Information (PHI).

Security & Compliance Tables:

Audit Logs: A centralized table that records every interaction. Note the fields for action_type, user_id, and timestamp which are mandatory for HIPAA.

MFA/Tokens: Tables to manage Two-Factor Authentication secrets and JWT refresh tokens.

Relationships:

One-to-Many: One Doctor to many Patients/Appointments.

Many-to-Many: Users to Roles (via a join table) to allow a user to be both a Doctor and an Admin.



Redis Caching: Minimizes database hits for static data.

Micrometer/Actuator: Monitors system health. If the CPU spikes or login failures increase, the system triggers alerts to prevent Denial of Service (DoS).
