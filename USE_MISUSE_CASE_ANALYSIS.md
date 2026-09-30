# Immich Use Case and Misuse Case Security Analysis

## Team Members

- Christian Theisen
- Adu
- Marc
- Matthew
- Dillon

---

## 1. Operational Context

This analysis builds upon the operational environment established in our project proposal for Immich. The system of interest is a self-hosted Immich deployment used by a five-person photography studio to manage, store, and share photos and videos.

### 1.1 System and Trust Boundaries

The security analysis considers interactions across the following system and trust boundaries:

- Internet
- Reverse proxy
- Immich application
- PostgreSQL database
- File storage
- External identity provider, where applicable

The PostgreSQL database and file storage are treated separately because the database contains application metadata and authentication-related information, while original photos and videos reside in file storage.

---

## 2. Essential Interactions and Use/Misuse Case Analysis

Five essential interactions between Immich and its operational environment were selected for use-case and misuse-case analysis.

### 2.1 Authentication and Login

**Owner:** Christian

#### Actors

**Photography Studio Member:** A member of the five-person photography studio who has an existing Immich account and uses the externally accessible Immich application to access the studio's photo-management functionality.

#### Use Case Description

The Authentication and Login use case represents a photography studio member accessing Immich using an existing account. The Photography Studio Member provides an email address and password through the Immich login interface. Immich validates the submitted credentials before granting authenticated access to the user's account and protected application functionality.

**Goal:** Authenticate to Immich and gain access to the member's authorized account and photo-management functionality.

**Precondition:** The Photography Studio Member has an existing Immich account with valid login credentials.

**Successful Outcome:** Immich validates the supplied credentials and establishes authenticated access for the Photography Studio Member.

#### Initial Use Case

The initial use-case analysis identified **Log In with Email and Password** as the primary interaction. The Photography Studio Member interacts with the Immich system through this use case to establish authenticated access.

The initial diagram intentionally represents only the legitimate interaction. Misusers, misuse cases, and security countermeasures are introduced during subsequent iterations of the analysis.

#### Misuser(s)

**External Credential Attacker:** An individual outside the photography studio who has network access to the studio's Internet-accessible Immich login interface. The attacker's motive is to obtain unauthorized access to a studio member's account and the photographs, metadata, or other resources available through that account. The attacker does not initially possess legitimate access to Immich but may obtain or discover a studio member's credentials through an external source, such as credential reuse or prior credential theft. The attacker can submit authentication requests through the same externally accessible login interface used by legitimate studio members.

#### Misuse Cases

**Gain Unauthorized Account Access:** The External Credential Attacker attempts to authenticate as a legitimate Photography Studio Member without authorization. This misuse case threatens the normal **Log In with Email and Password** use case.

**Use Stolen or Reused Credentials:** The External Credential Attacker submits credentials belonging to a legitimate Photography Studio Member. Because the supplied credentials may be valid, ordinary credential validation alone may not distinguish the attacker from the legitimate account owner. This misuse case therefore threatens the **Validate Login Credentials** security functionality.

#### Security Countermeasures

**Validate Login Credentials:** Immich validates the email address and password supplied during authentication before granting authenticated access. This security functionality mitigates attempts to gain unauthorized account access when an attacker supplies invalid credentials.

The analysis also identified a limitation of credential validation. If an attacker obtains valid credentials belonging to a Photography Studio Member, successful password validation alone cannot determine whether the person submitting those credentials is the legitimate account owner. This limitation motivated an additional misuse case rather than assuming that credential validation completely prevents unauthorized access.

#### Iterative Analysis

The Authentication and Login analysis was developed recursively by introducing threats and security functionality across multiple iterations.

**Initial Use Case:** The analysis began with the Photography Studio Member interacting with **Log In with Email and Password**. This established the legitimate interaction between an external actor and Immich.

**Iteration 1 – Unauthorized Account Access:** An **External Credential Attacker** and the misuse case **Gain Unauthorized Account Access** were introduced. The misuse case threatens the legitimate login use case and represents an external attacker attempting to authenticate as a studio member without authorization.

**Iteration 2 – Credential Validation:** The security functionality **Validate Login Credentials** was introduced. The **Log In with Email and Password** use case includes credential validation, which mitigates unauthorized-access attempts involving invalid credentials.

**Iteration 3 – Stolen or Reused Credentials:** The analysis was repeated against the newly identified security functionality. The misuse case **Use Stolen or Reused Credentials** was introduced because an attacker possessing valid credentials may successfully satisfy ordinary credential validation. This misuse case therefore threatens **Validate Login Credentials** and demonstrates that password validation does not completely eliminate the risk of account compromise.

The final diagram incorporates the legitimate actor, contextualized misuser, identified misuse cases, and security functionality discovered through this iterative process.

#### Final Use/Misuse Case Diagram

The final diagram incorporates the legitimate Authentication and Login interaction, the External Credential Attacker, the identified misuse cases, and the security functionality derived through iterative analysis.

![Authentication and Login Final Use/Misuse Case Diagram](authentication-login-final.drawio.png)

#### Derived Security Requirements

The iterative misuse-case analysis produced the following functional security requirements for Authentication and Login:

- **SR-AUTH-01 – Credential Validation:** Immich shall validate the email address and password supplied by a user before establishing an authenticated session or granting access to protected application functionality.

- **SR-AUTH-02 – Authentication Failure:** Immich shall deny authentication when the supplied email address or password cannot be successfully validated.

- **SR-AUTH-03 – Authentication Failure Disclosure:** Immich shall avoid revealing whether an authentication failure resulted from an invalid email address or an invalid password, reducing information available for account-enumeration attacks.

- **SR-AUTH-04 – Authentication Event Logging:** Immich shall record failed authentication attempts with sufficient information to support security monitoring and investigation.

- **SR-AUTH-05 – Compromised Credential Risk:** Authentication controls should provide a means of reducing reliance on reusable application passwords because possession of a valid studio member's password may allow an unauthorized user to satisfy ordinary password validation.

#### Immich Implementation Evidence

The derived Authentication and Login requirements were compared against the current Immich implementation and official documentation.

| Requirement | Support | Immich Implementation Evidence |
| --- | --- | --- |
| **SR-AUTH-01** | Supported | Immich's authentication service retrieves the user by email and compares the submitted password with the stored password hash using bcrypt. A successful authentication results in creation of the login response and session. |
| **SR-AUTH-02** | Supported | If the user does not exist, has no password, or the password comparison fails, Immich rejects the request with an unauthorized response rather than establishing a session. |
| **SR-AUTH-03** | Supported | Immich returns the same "Incorrect email or password" response for unsuccessful password authentication. It also performs a bcrypt comparison against a dummy hash when an email is not registered to reduce timing-based user enumeration. |
| **SR-AUTH-04** | Supported | Immich records a warning for a failed login attempt containing the submitted email address and source IP address. |
| **SR-AUTH-05** | Partially Supported | Immich supports third-party authentication through OpenID Connect (OIDC) on top of OAuth2 and allows administrators to disable local password authentication. This provides a deployment option that can reduce reliance on Immich-managed reusable passwords. However, ordinary email/password authentication still relies on possession of valid credentials and does not by itself distinguish a legitimate user from an attacker who has obtained those credentials. |

Overall, Immich directly supports the credential-validation, authentication-failure, failure-disclosure, and failed-login logging requirements identified by this analysis. The compromised-credential requirement is only partially addressed by the availability of OAuth/OIDC and the ability to disable password authentication. The strength of authentication provided by an external identity provider depends on that provider's configuration and security controls.

---

### 2.2 Upload and Private Asset Access

**Owner:** Matthew

#### Actors

TBD

#### Use Case Description

TBD

#### Initial Use Case

TBD

#### Misuser(s)

TBD

#### Misuse Cases

TBD

#### Security Countermeasures

TBD

#### Iterative Analysis

TBD

#### Final Use/Misuse Case Diagram

*Diagram to be added.*

#### Derived Security Requirements

TBD

#### Immich Implementation Evidence

TBD

---

### 2.3 Shared Albums and Public Links

**Owner:** Dillon

#### Actors

TBD

#### Use Case Description

TBD

#### Initial Use Case

TBD

#### Misuser(s)

TBD

#### Misuse Cases

TBD

#### Security Countermeasures

TBD

#### Iterative Analysis

TBD

#### Final Use/Misuse Case Diagram

*Diagram to be added.*

#### Derived Security Requirements

TBD

#### Immich Implementation Evidence

TBD

---

### 2.4 Administrative Access and User Management

**Owner:** Marc

#### Actors

**Photography Studio Administrator:** An administrative member of the five-person photography studio who has an existing Immich account and has the responsibility for administering the studio's Immich application. They can create accounts when employees join, reset passwords, configure user storage, and remove accounts when employees leave.

#### Use Case Description

The **Administrative Access and User Management** use case represents a photography studio administrator managing the application through their account with elevated privileges. Using the administrative functions to manage user access to Immich within the photography studio, the designated Photography Studio Administrator is responsible for managing the accounts of the five staff members. The administrator can create new user accounts, modify existing user information, reset user passwords, configure user storage quotas, and remove users who should no longer have access to the system.

These administrative functions are important to the studio because user accounts control access to client photos, albums, and other potentially sensitive information stored within Immich. Administrative user management functions should only be available to an authenticated user with the appropriate elevated administrator privileges. Regular studio users may access the Immich features permitted by their basic accounts but should not be able to perform administrative actions affecting other users.

#### Initial Use Case

The initial use case focuses on the interaction between the **Photography Studio Administrator** and Immich's functionality to manage user accounts. The administrator accesses the administrative suite to manage the accounts of studio staff members, including creating accounts for new users, modifying existing accounts, resetting passwords, configuring storage quotas, and removing users who no longer require access.

In the initial use case diagram, the **Photography Studio Administrator** is the primary actor and **Administrative Access and User Management** is the use case within the Immich system. At this point, the diagram represents only the intended administrative interaction and does not yet include any potential misuse or security threats.

#### Misuser(s)

**Unauthorized Studio Member:** A photography studio employee who has a valid Immich account but does not have administrator privileges. This user has legitimate access to normal Immich functionalities but may attempt to gain unauthorized access to administrative functions. Their goal could include modifying another user's account, resetting passwords, creating unauthorized accounts, or deleting existing users.

**Compromised-Admin Attacker:** An unauthorized individual who has obtained access to a valid **Photography Studio Administrator** account or authenticated administrative session. Because the attacker is operating through an account with legitimate administrator privileges, they may attempt to abuse Immich's user-management functionality to modify accounts, reset passwords, create unauthorized users, or remove legitimate studio users.

#### Misuse Cases

**Perform Unauthorized User Administration:** An **Unauthorized Studio Member** attempts to access administrator-only user management functions despite having a non-administrative account. The user may attempt to create new accounts, modify other users, reset another user's password, or delete existing accounts. This misuse case threatens the **Administrative Access and User Management** use case because successful unauthorized access could allow a regular studio employee to alter who can access Immich and the studio's client data.

**Abuse Compromised Administrator Access:** A **Compromised-Admin Attacker** uses access to a valid administrator account or authenticated administrative session to perform unauthorized user management actions. Because the compromised account possesses legitimate administrator privileges, the attacker may be able to create unauthorized accounts, modify existing users, reset passwords, or remove legitimate studio users. This misuse case threatens the **Administrative Access and User Management** use case by abusing legitimate administrative privileges rather than attempting to access the functionality through a non-administrative account.

#### Security Countermeasures

**Verify Administrator Authorization:** Immich verifies that an authenticated user has administrator privileges before allowing access to administrator-only user management functionality. A regular studio member may have a valid Immich account, but authentication alone does not authorize the user to perform administrative actions. Immich must also verify the user's administrator status before allowing actions such as creating, modifying, or deleting other user accounts. This countermeasure mitigates **Perform Unauthorized User Administration**.

**Deny Unauthorized Administrative Requests:** If an authenticated user without administrator privileges attempts to access an administrator-only function, Immich denies the request. This server-side authorization enforcement prevents a regular studio member from gaining administrative capabilities simply by attempting to directly access an administrative endpoint or bypass the normal user interface. This countermeasure further mitigates **Perform Unauthorized User Administration**.

These authorization controls provide protection against users who lack administrator privileges. However, they do not fully mitigate **Abuse Compromised Administrator Access** because an attacker using a valid administrator account or authenticated administrator session may satisfy Immich's administrator authorization checks. This limitation requires further consideration during the iterative misuse-case analysis.

#### Iterative Analysis

The initial misuse-case analysis identified that an Unauthorized Studio Member could attempt to access administrator-only user management functions. To address this misuse, **Verify Administrator Authorization** requires Immich to verify that the authenticated user has administrator privileges before allowing administrative actions. **Deny Unauthorized Administrative Requests** further protects the use case by rejecting administrative requests made by users who do not have the required privileges.

After introducing these countermeasures, the analysis was repeated to determine whether the Manage User Accounts use case could still be misused. This identified **Abuse Compromised Administrator Access**. If an attacker obtains access to a legitimate administrator account or authenticated administrator session, the attacker may pass through Immich's administrator authorization checks and gain access to the same user management functions available to the actual administrator.

This demonstrates that administrator authorization protects against authenticated users who lack the required privileges, but it depends on the security of the administrator's authentication and session. Immich's existing authentication and session controls therefore provide an additional layer of protection by requiring a valid authenticated session before administrator authorization is evaluated. The final use/misuse case diagram reflects both the initial authorization threat and the additional risk created when valid administrator access is compromised.

#### Final Use/Misuse Case Diagram

<img width="936" height="852" alt="administrative_access_user_management drawio" src="https://github.com/user-attachments/assets/b027c02b-0540-4099-8239-d0f3b23cdab5" />


#### Derived Security Requirements

Based on the use/misuse case analysis, the following security requirements were derived for administrative access and user management:

**SR-ADMIN-01 – Administrator Authorization:** Immich shall verify that an authenticated user has administrator privileges before permitting access to administrator-only user management functions.

**SR-ADMIN-02 – Unauthorized Administrative Access Prevention:** Immich shall deny requests to perform administrator-only user management operations when the authenticated user does not possess administrator privileges.

**SR-ADMIN-03 – Authentication and Session Validation:** Immich shall require a valid authenticated user session before evaluating and permitting access to administrative user management functionality.

**SR-ADMIN-04 – Administrative Function Restriction:** Immich shall restrict security-sensitive user management operations, including creating users, modifying user accounts, resetting user passwords, and deleting users, to authorized administrators.

#### Immich Implementation Evidence

| Requirement | Support | Immich Implementation Evidence |
| --- | --- | --- |
| **SR-ADMIN-01** | Supported | AuthService.authenticate() checks the authenticated user's isAdmin property when an administrator-only route is requested. |
| **SR-ADMIN-02** | Supported | Immich logs denied access and returns Forbidden when a non-administrator attempts to access an administrator-only route. |
| **SR-ADMIN-03** | Supported | Immich validates authentication before performing the administrator authorization check and maintains server-side session records associated with authenticated users.|
| **SR-ADMIN-04** | Supported | Immich documents administrator functionality for creating users, resetting passwords, configuring user storage, and deleting users. |

---

### 2.5 OAuth/OIDC and External Authentication

**Owner:** Adu

#### Actors

TBD

#### Use Case Description

TBD

#### Initial Use Case

TBD

#### Misuser(s)

TBD

#### Misuse Cases

TBD

#### Security Countermeasures

TBD

#### Iterative Analysis

TBD

#### Final Use/Misuse Case Diagram

*Diagram to be added.*

#### Derived Security Requirements

TBD

#### Immich Implementation Evidence

TBD

---

## 3. Consolidated Security Requirements

The following table consolidates the functional security requirements derived from the five misuse-case analyses and maps them to the threats and security controls that motivated them.

| ID | Security Requirement | Derived From | Security Property / Control | Immich Support | Evidence |
|---|---|---|---|---|---|
| SR-AUTH-01 | TBD | Authentication/Login | TBD | TBD | TBD |
| SR-ASSET-01 | TBD | Private Asset Access | TBD | TBD | TBD |
| SR-SHARE-01 | TBD | Shared Links | TBD | TBD | TBD |
| SR-ADMIN-01 | TBD | Administrative Access | TBD | TBD | TBD |
| SR-OIDC-01 | TBD | OAuth/OIDC | TBD | TBD | TBD |

---

## 4. AI-Assisted Use/Misuse Case Analysis

### 4.1 Prompt Used

TBD

### 4.2 Suggestions Produced

TBD

### 4.3 Improvements Made to the Diagrams

TBD

### 4.4 Usefulness and Limitations

TBD

---

## 5. Alignment with Immich Security Features

### 5.1 Supported Security Requirements

TBD

### 5.2 Partially Supported Security Requirements

TBD

### 5.3 Identified Security Gaps

TBD

### 5.4 Sufficiency of Existing Security Features

TBD

---

## 6. Security Configuration and Installation Documentation Review

### 6.1 Existing Security Guidance

TBD

### 6.2 Missing or Unclear Security Guidance

TBD

### 6.3 Recommended Documentation Improvements

TBD

---

## 7. Project Management and Collaboration

### 7.1 GitHub Project Board

[Immich Project Board](ADD_PROJECT_BOARD_LINK_HERE)

### 7.2 Team Task Assignments

| Team Member | Primary Responsibility | Secondary Responsibility |
|---|---|---|
| Christian | Authentication/Login | Report integration and project board |
| Matthew | Upload and Private Asset Access | Implementation verification |
| Dillon | Shared Albums and Public Links | Security requirements compilation |
| Marc | Administrative Access and User Management | Diagram consistency review |
| Adu | OAuth/OIDC and External Authentication | Security documentation review |

### 7.3 Team Collaboration

TBD

---

## 8. Team Reflection

### Christian Theisen

For this assignment, I was responsible for the Authentication and Login use/misuse case analysis and helped organize the team's work through the GitHub Project Board. I created the project issues for the five essential interactions and used a separate branch and pull request for my analysis so that the work could be reviewed before being merged into the main report.

My analysis began with the legitimate interaction between a Photography Studio Member and Immich's email/password login functionality. I then iteratively introduced an External Credential Attacker, unauthorized account access, credential validation, and the threat created by stolen or reused credentials. One of the most useful parts of this process was recognizing that a security control can itself become the subject of another misuse case. Although validating credentials mitigates attempts involving incorrect credentials, it cannot by itself distinguish a legitimate user from an attacker who possesses valid stolen credentials.

I also compared the security requirements derived from the analysis with Immich's current source code and official documentation. This helped distinguish between requirements that Immich directly supports and security needs that are only partially addressed. In particular, the analysis showed the importance of considering both software controls and the limitations of password-based authentication rather than assuming that successful credential validation completely resolves authentication threats.

From a teamwork and project-management perspective, I focused on making our work easier to track and review by creating GitHub issues, organizing tasks on the project board, and submitting my work through a pull request with a requested teammate review. This workflow improved the visibility of individual contributions and provided an opportunity for feedback before integration into the final report.

### Adu Peprah

<img width="1028" height="597" alt="image_001" src="https://github.com/user-attachments/assets/f2b708f7-cee9-4bdf-b314-9e08ac992978" />




Team 6 project proposal on open source project decided to evaluate on security vulnerability of an application called immich. Immich is a high-performance, self-hosted photo and video backup solution designed to be a complete, private alternative to cloud services like Google Photos and Apple iCloud. It has gained massive popularity in the self-hosting community because it replicates the slick user experience, speed, and modern features of commercial tech giants while keeping all data entirely on your own hardware.

I was assigned to work on OAuth/OIDC/External Authentication.I reviewed Immich features built-in support for third-party authentication using OpenID Connect (OIDC), an identity layer built on top of OAuth 2.0. This allows you to integrate Immich with popular self-hosted and enterprise identity providers (IdPs) like Authentik, Authelia, Keycloak, Pocket ID, Okta, or public providers like Google. I reviewed Immich authentication on security vulnerability and in my submission created a USE CASE and recommendation to avert. Links Below on details of my submission. 
Details of my contribution can be seen on Github via links below

1.	OAuth/OIDC/External Authentication : https://github.com/apeprah1/Immich-OSS-Project-Proposal-/blob/OSS-Project-Proposal--Immich/Group%206%20OAuth%20OIDC%20External%20Authentication_images/image_001.png

https://github.com/apeprah1/Immich-OSS-Project-Proposal-/blob/OSS-Project-Proposal--Immich/Group%206%20OAuth%20OIDC%20External%20Authentication_images/Group%206%20OAuth%20OIDC%20External%20Authentication.md

2.	Security documentation review :
 https://github.com/apeprah1/Immich-OSS-Project-Proposal-/blob/OSS-Project-Proposal--Immich/Group%206%20%20Immich%20Security%20Document%20Review_images/Group%206%20%20Immich%20Security%20Document%20Review.md



TBD

### Marc

TBD

### Matthew

TBD

### Dillon

TBD

### 8.1 Combined Team Reflection

TBD

---

## References

- Immich. "Auth Service." *Immich GitHub Repository*.  
  https://github.com/immich-app/immich/blob/main/server/src/services/auth.service.ts

- Immich. "OAuth." *Immich Documentation*.  
  https://docs.immich.app/administration/oauth/

- Immich. "System Settings." *Immich Documentation*.  
  https://docs.immich.app/administration/system-settings/
