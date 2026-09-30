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

**Studio Member:** A member of the photography studio who has an authenticated Immich account and uses the Immich mobile or web app to upload photos from client shoots and to view, download, and organized any assets they own. Studio members rely on Immich to keep unfinished assets and personal photos visible only to the uploading member until that member shares them intentionally. 

#### Use Case Description

The Upload and Private Asset Access use case represents a Photography Studio Member adding new photos to Immich and then accessing their own private media. The member uploads photos/videos through the Immich client, Immich then stores the data, records the member as the owner of the asset, and processes it in the background. The studio member can then view, download, and organize their assets, and move any sensitive assets into a locked folder.

**Goal:** Upload photos to Immich and access assets owned by the requesting authenticated member while preventing access by other users of the same instance.

**Precondition:** The Studio Member is authenticated to Immich with an active session. 

**Successful Outcome:** Immich stores the uploaded asset under the member's ownership and returns only assets the member is authorized to access.

#### Initial Use Case

Initial use-case analysis identified three main interactions: **Upload Asset, View and Download Owned Assets, and Move Assets to Locked Folder.** The Studio Member interacts with Immich for all three.

#### Misuser(s)

**Curious Studio Employee:** An employee that holds a legitimate authenticated Immich account on the same instance. The employee's motive is to view a colleague's unreleased work or personal photos out of curiosity or competitive interest. 

Because the employee is already successfully authenticated, credential-based access controls don't apply in this situation. The Employee's resources are simple: A web broswer, developer tools, and Immich's REST API, which is publicly documented. 

The attack of choice is to bypass the client interface and request asset identifiers belonging to another member, assuming that the app filters content only in the user interface. 

**Opportunistic Device Holder:** Someone that has obtained a studio member's phone that was either lost or stolen. The motive is to access private photos or videos for extortion or resale. The device holder has no credentials but has an active Immich session on the downloaded mobile app, which gives them the same access as the legitimate owner of the account. 

The attack of choice is to simply open the app and browse/download the legitimate member's assets, including whatever assets the user thought were protected by the Locked Folder.

#### Misuse Cases

**Access Another Member's Private Assets:** The curious studio employee requests assets owned by a different studio member by providing that member's asset identifiers directly to the API. This misuse case threatens the View and Download Own Assets use case.

**Use an Active Session on a Stolen Device:** The opportunistic device holder goes through a session that Immich considers legitimate. Because the request has the legitimate owner's identity, ownership-based authorization can't tell the device holder from the legitimate member, this case threatens the Enforce Asset Ownership security functionality. 

**Bypass Locked Folder Protection:** Either of the misusers attempts to reach assets that are marked as locked without satisfying the PIN requirement.

For example, they could do so by requesting an endpoint that returns assets without applying the locked-visibility restriction. This case threatens the Require PIN-Elevated Session security functionality. 

#### Security Countermeasures

**Assign Ownership at Upload:** Immich assigns the authenticated uploader as the owner of each newly created asset rather than accepting an owner identifier from the client. This establishes the ownership relationship that all authorization decisions later on depend on.

**Enforce Asset Ownership:** Immich does a server-wide authorization check for any requested asset identifier, confirming that the authenticated user owns or has been granted access to the asset. This mitigates attempts to access another member's assets through identifiers. 

Analysis also found a limitation of ownership enforcement. Because the authorization check resolves the identity carried by the session, it can't distinguish between the legitimate owner from someone that possesses the owner's device. 

**Require PIN-Elevated Session:** Assets stored in the Locked Folder can be accessed only by a session that has been elevated by entering the account PIN, and that elevation expires after a timed period, introducing a second authorization factor inside of an already-authenticated account.

**Revoke Active Sessions:** Immich allows users to list the sessions associated with their account and devices and revoke them individually or all at once, so a lost device can be cut off without having to change account credentials. 

#### Iterative Analysis

**Initial Use Case:** Analysis began with the Photography Studio Member interacting with Upload Asset, View and Download Owned Assets, and Move Assets to Locked Folder.

**Iteration 1 - Insider Access to Private Assets:** The Curious Studio Employee and the misuse case Access Another Member's Private Assets were introduced. This misuse case threatens the legitimate asset-access use case.

**Iteration 2 - Ownership Enforcement:** Security functions Assign Ownership at Upload and Ownership Enforcement were introduced. Upload Asset includes the ownership assignment and View and Download Owned Assets includes the ownership check, mitigating insider access using identifiers.

**Iteration 3 - Stolen Device Sessions:** Analysis was repeated against the newly identified security functionality. The opportunistic device holder and the misuse case Use an Active Session on a Stolen Device were introduced because a request made from the legitimate owner's device will satisfy the ownership enforcement. This case threatens Enforce Asset Ownership and shows that ownership checking won't protect assets if a session is compromised. 

**Iteration 4 - Session Revocation and PIN Elevation:** Revoke Active Sessions and Require PIN-Elevated Session were introduced to mitigate the stolen session misuse case. These countermeasures allow a member to terminate a compromised session and further secure their most sensitive assets. 

**Iteration 5 - Locked Folder Bypass:** Analysis was repeated against PIN control. The misuse case Bypass Locked Folder Protection was introduced because the locked-visibility restriction should be applied consistently by every endpoint that returns assets. If any endpoint does not apply the restriction, a misuser can reach locked assets from a session that never entered the account PIN. This case threatens Require PIN-Elevated Session and created a requirement for default-deny enforcement instead of per-endpoint enforcement. 

The final diagram incorporates the legitimate actor, the two misusers, the three misuse cases, and the four security functions discovered through the iterative analysis. 

#### Final Use/Misuse Case Diagram

![Upload and Private Asset Access Final Use/Misuse Case Diagram](private-asset-access-final.drawio.png)

#### Derived Security Requirements

The iterative analysis produced these security requirements for Upload and Private Asset Access:

**SR-ASSET-01 - Ownership Assignment:** Immich will assign the authenticated uploading user as the owner of each newly uploaded asset and will not accept a client provided ownership identifier.

**SR-ASSET-02 - Server-Side Authorization:** Immich will verify for every requested asset identifier that the authenticated user is authorized to access the asset before providing any metadata, thumbnails, or original files regardless of any filtering done by the client.

**SR-ASSET-03 - Authorization Failure:** Immich will deny requests for any assets the user is not authorized to access, without telling if the requested identifier exists or not.

**SR-ASSET-04 - Session Revocation:** Immich will allow a user to view and revoke any active sessions associated with their account, so that access from a lost or stolen device can be terminated.

**SR-ASSET-05 - Secondary Authorization for Sensitive Assets:** Immich will restrict access to assets marked as locked to sessions that have been elevated using the account PIN, and will expire that elevation after a set period.

**SR-ASSET-06 - Consistent Visibility Enforcement:** Immich will apply the locked-visibility restriction by default across all endpoints that return assets, instead of relying on each endpoint to set the restriction individually. 

#### Immich Implementation Evidence

| Requirement | Support | Implementation Evidence |
| ---------- | ---------- | ----------|
| SR-ASSET-01 | Supported | Immich sets the authenticated uploader as the asset owner at creation; ownership is the same in postgreSQL, which is where information on access/authorization, users, and assets is stored. |
| SR-ASSET-02 | Supported | Authorization is centralized in server/src/utils/access.ts. Controllers then call requireAccess with a permission and requested IDs resolved against repositories/access.repository.ts |
| SR-ASSET-03 | Supported | The access check runs before the request handler, so unauthorized identifiers are rejected before they are confirmed. |
| SR-ASSET-04 | Supported | Sessions are listed and revocable per device in the user settings. Release v3.1.0 added the option to invalidate sessions on password reset |
| SR-ASSET-05 | Supported | The Locked Folder is advertised as extra protection for sensitive media, it uses a server-side PIN and session elevation. |
| SR-ASSET-06 | Partially Supported | Elevation is enforced per handler. A study done in 2026 found that 4 out of 5 of the endpoints enforced the PIN requirement, while POST /search/random didn't. leaving out the visibility field returned assets that were supposed to be locked to a session that never entered the PIN. |

Immich supports the Ownership, Authorization, Authorization Failure, Session Revocation, and PIN-Elevation. Consistent Visibility Enforcement is only partially addressed and supported. Ownership checks are centralized but the visibility restriction is applied individually, endpoint to endpoint. 

---

### 2.3 Shared Albums and Public Links

**Owner:** Dillon

#### Actors

**Photography Studio Member:** A staff member at the photography studio who uses Immich to organize photos and create public shared links for delivering selected photos to clients.

**Client:** A customer of the photography studio who receives a public shared link and uses it to access photos intentionally shared by the studio.

#### Use Case Description

The Shared Albums and Public Links use case represents a Photography Studio Member using Immich to share selected photos with a client through a public link. The Studio Member selects the photos that should be shared and creates a public link that can be sent to the Client. The Client can then use the link to access the shared photos without needing an Immich account.

**Goal:** Allow a Photography Studio Member to securely share selected photos with a Client without exposing other private photos stored in Immich.

**Precondition:** The Photography Studio Member is logged into Immich and has access to the photos that will be shared.

**Successful Outcome:** Immich creates a public share that allows the Client to access the intended photos while following the sharing restrictions set by the Studio Member.

#### Initial Use Case

The initial use-case analysis identified **Create Public Share** and **Access Shared Photos** as the primary interactions. The Photography Studio Member creates a public share containing selected photos, and the Client uses the resulting link to access the shared content.

The initial analysis represents the intended sharing interaction before misuse cases and security countermeasures are introduced.

#### Misuser(s)

**Unauthorized Shared-Link Holder:** Someone who gets access to a public shared link even though the photos were not meant for them. The link could have been forwarded to them or exposed in some other way. They could then try to use the link to view photos without permission. This person does not have a studio account but can still reach Immich's public sharing page through the link.

#### Misuse Cases

**Access Shared Content Without Authorization:** An Unauthorized Shared-Link Holder gets a public link that was meant for a Client and tries to use it to view the shared photos. This threatens the normal **Access Shared Photos** use case because someone who was not supposed to receive the link could potentially use it to view the content.

**Access Photos Using Expired Link:** An Unauthorized Shared-Link Holder tries to access the shared photos after the link was supposed to expire. This also threatens **Access Shared Photos** because a link that is no longer valid should not continue giving someone access to the photos.

#### Security Countermeasures

**Password-Protected Shared Link:** Immich allows the Studio Member to add a password to a public shared link. This adds another layer of protection because having the link by itself is not enough to access the photos. The person opening the link also needs to provide the correct password.

**Set Share Expiration:** Immich allows the Studio Member to set an expiration time for a public shared link. Once that time has passed, the link should no longer allow access to the shared photos. This helps prevent old links from continuing to provide access longer than the Studio Member intended.

#### Iterative Analysis

The Shared Albums and Public Links analysis was built by starting with the normal sharing process and then looking at ways that process could be misused.

**Initial Use Case:** The analysis started with the Photography Studio Member creating a public share and the Client using the link to access the selected photos. This represents how the sharing feature is supposed to work normally.

**Iteration 1 – Unauthorized Shared Access:** The **Unauthorized Shared-Link Holder** was introduced along with the **Access Shared Content Without Authorization** misuse case. This represents someone getting a shared link that was not intended for them and attempting to use it to view the photos.

**Iteration 2 – Password Protection:** The **Password-Protected Shared Link** security feature was added as a countermeasure. Requiring a password gives the shared link another layer of protection so that simply getting the link may not be enough to access the photos.

**Iteration 3 – Expired Link Access:** The **Access Photos Using Expired Link** misuse case was then added to represent someone attempting to continue using a shared link after the Studio Member intended for access to end.

**Iteration 4 – Share Expiration:** The **Set Share Expiration** security feature was added to address this misuse case. Setting and enforcing an expiration time limits how long the shared link can continue to provide access.

The final diagram shows the normal sharing process along with the two misuse cases and the security features that help address them.

#### Final Use/Misuse Case Diagram

The final diagram shows the Photography Studio Member and Client using Immich's public sharing feature, along with the Unauthorized Shared-Link Holder, the two identified misuse cases, and the security features added to address those threats.

<img width="835" height="552" alt="image" src="https://github.com/user-attachments/assets/01cc69aa-5274-44a6-992b-9c55920e0886" />

#### Derived Security Requirements

The misuse-case analysis produced the following security requirements for Shared Albums and Public Links:

- **SR-SHARE-01 – Shared-Link Password Protection:** Immich shall require the correct password before allowing access to a password-protected public share.

- **SR-SHARE-02 – Invalid Shared-Link Password:** Immich shall deny access when an incorrect password is provided for a password-protected public share.

- **SR-SHARE-03 – Shared-Link Expiration:** Immich shall prevent access to a public shared link after its configured expiration time has passed.

- **SR-SHARE-04 – Expiration Validation:** Immich shall check whether a public shared link has expired before allowing access to the photos associated with that link.

#### Immich Implementation Evidence

Immich already includes several protections that line up with the security requirements identified in this analysis. The public sharing documentation and source code were reviewed to see how each requirement is handled.

| Requirement | Status | How Immich Addresses It |
|---|---|---|
| SR-SHARE-01 | Implemented | Public shared links in Immich can be protected with a password. When password protection is enabled, the correct password must be provided before the shared content can be accessed. |
| SR-SHARE-02 | Implemented | Immich checks the password submitted for a protected share. If the password is incorrect, access to the share is rejected. |
| SR-SHARE-03 | Implemented | A public share can be given an expiration date. After that expiration time is reached, the shared link is no longer treated as valid. |
| SR-SHARE-04 | Implemented | When a shared link is validated, Immich checks its expiration information. The link remains valid if no expiration was set or if the expiration time has not been reached. |

Overall, the protections identified in the misuse-case analysis are already represented in Immich. Password protection helps limit access when a link reaches someone it was not intended for, while expiration dates limit how long a shared link can continue providing access. Together, these features address the two misuse cases identified in the diagram.

---

### 2.4 Administrative Access and User Management

**Owner:** Marc

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

### 2.5 OAuth/OIDC and External Authentication

**Owner:** Adu

#### Actors

<img width="1028" height="597" alt="image_001" src="https://github.com/user-attachments/assets/dedec495-f531-40af-9a12-951db678af32" />

User (Web or Mobile Client): The primary actor. The user wants to access their Immich account using external authentication.
External Identity Provider (IdP): The external system responsible for authenticating the user. Depending on an Immich deployment, this can be an OIDC-compatible identity provider. Its role is to verify the user's identity and return authentication information to Immich.
Main use case: Authenticate with OAuth/OIDC
The central use case is Authenticate with OAuth/OIDC. Instead of entering credentials that Immich itself validates, the user is redirected to the configured external identity provider.

The Immich server is the system of interest. It initiates the external authentication flow, processes the OAuth callback, validates OIDC identity information, creates or links the user's account when necessary, and establishes the authenticated session.

#### Use Case Description

The primary use case is Authenticate with OAuth/OIDC. Instead of entering credentials that Immich validates directly, the user chooses external authentication and is redirected to the configured IdP.
The basic interaction is:
User → Immich → External Identity Provider → Immich → User
The user selects external login, Immich sends an authorization request to the IdP, the IdP authenticates the user, and the IdP returns authorization information to Immich through the configured callback. Immich then validates the returned identity, creates or links the local account when required, and establishes an authenticated Immich session

#### Initial Use Case

The initial legitimate authentication flow is:
1. Initiate External Login — The user selects OAuth/OIDC authentication.
2. Authenticate with OAuth/OIDC — Immich generates an authorization request and redirects the user to the configured IdP.
3. Authenticate User at IdP — The external IdP authenticates the user. Depending on IdP configuration, this could involve a password, MFA, passkey, or another authentication mechanism.
4. Handle OAuth Callback — The IdP redirects the user back to Immich with the authorization response.
5. Validate OIDC Identity — Immich validates the returned authentication information and obtains the identity necessary to identify the user.
6. Create/Link User Account — For an initial external login, Immich may associate the external identity with a new or existing Immich account. This is conditional and therefore modeled using <<extend>>.
7. Establish Immich Session — After successful authentication and account association, Immich creates the authenticated session.

#### Misuser(s)

| Misuser | Objective |
|---|---|
| **Remote Attacker** | Obtain unauthorized access to another user's Immich account. |
| **Network/MITM Attacker** | Intercept or manipulate communications between Immich and the IdP. |
| **Malicious OAuth/OIDC Client/User** | Manipulate callbacks, state values, authorization codes, or redirect parameters. |
| **Malicious/Compromised Identity Provider** | Supply fraudulent identity or privilege claims to Immich. |
| **Authenticated Malicious User** | Improperly link an external identity to another account or exploit externally supplied profile data. |

#### Misuse Cases
| Misuser | Objective |
|---|---|
| **Remote Attacker** | Obtain unauthorized access to another user's Immich account. |
| **Network/MITM Attacker** | Intercept or manipulate communications between Immich and the IdP. |
| **Malicious OAuth/OIDC Client/User** | Manipulate callbacks, state values, authorization codes, or redirect parameters. |
| **Malicious/Compromised Identity Provider** | Supply fraudulent identity or privilege claims to Immich. |
| **Authenticated Malicious User** | Improperly link an external identity to another account or exploit externally supplied profile data. |


#### Security Countermeasures

The misuse cases introduce the following countermeasures:
1. OAuth state validation — protect authentication transactions from callback manipulation and login CSRF.
2. PKCE (Proof Key for Code Exchange) — prevent a stolen authorization code from being sufficient to complete authentication.
3. OIDC identity/token validation — validate issuer, signatures, trusted signing keys, authorization transaction, and identity claims.
4. Controlled account linking — associate identities only after successful validation and prevent silent replacement of existing OAuth associations.
5. Configurable automatic registration — permit administrators to disable automatic account creation.
6. Validated role claims — accept only recognized authorization roles from a validated IdP.
7. Strict TLS certificate validation — authenticate the IdP during OIDC discovery, token, user-info, and JWKS communications.
8. External URL/network destination validation — prevent OAuth profile data from causing SSRF against protected resources.

#### Iterative Analysis
The analysis proceeds through successive attacker adaptations and corresponding security controls.

Iteration 1: Callback Manipulation → OAuth State Validation.
The initial authentication flow is susceptible to an attacker submitting an OAuth callback that does not belong to the authentication transaction initiated by the legitimate user. This introduces OAuth state validation. Immich should generate an unpredictable state value and reject a callback when the expected state is missing or invalid

Iteration 2: Authorization-Code Theft → PKCE.
State validation does not address an attacker who obtains the authorization code itself. PKCE therefore introduces a code_verifier/code_challenge relationship so that possession of the authorization code alone is insufficient to complete the authentication transaction.

Iteration 3: Forged Identity → OIDC Identity Validation.
An attacker may instead manipulate identity information. Immich must therefore cryptographically validate the returned OIDC identity, issuer, signatures, signing keys, and authentication transaction before using claims for account identification or authorization.

Iteration 4: Account-Linking Attack → Controlled Account Linking.
Even correctly authenticated identity information creates a risk when it is mapped to a local account. Account linking must occur only after successful identity validation, and an existing OAuth association must not be silently overwritten

Iteration 5: Unauthorized Provisioning → Disable Auto-Registration.
A legitimate external identity should not automatically imply authorization to become an Immich user. Administrators therefore require control over automatic account registration.

Iteration 6: Role Manipulation → Trusted Role Claims.
An attacker or compromised IdP may attempt to supply an administrator role. Role claims must therefore come from the successfully validated IdP and be restricted to recognized Immich role values

Iteration 7: IdP Impersonation → TLS Validation.
Even correct token-processing logic can be undermined if an attacker can impersonate the IdP. Immich must therefore perform strict TLS certificate validation for OIDC communications. The document notes that this corresponds to a real Immich vulnerability that was subsequently corrected.

Iteration 8: Malicious Profile Resource → SSRF Protection.
Finally, successfully authenticated profile information can itself contain attacker-controlled URLs. External resource URLs must therefore be validated before Immich performs server-side requests.


#### Final Use/Misuse Case Diagram

*Diagram to be added.*

#### Derived Security Requirements

SR-OIDC-01 — OAuth State Validation:
The Immich authentication system shall generate and validate an unpredictable OAuth state value for each external authentication transaction and shall reject callbacks for which the expected state is missing or invalid.

SR-OIDC-02 — PKCE:
The Immich authentication system shall use PKCE for OAuth/OIDC authorization-code authentication and shall reject the authentication transaction if the required PKCE code verifier is missing or invalid.

SR-OIDC-03 — OIDC Identity Validation:
The Immich authentication system shall cryptographically validate identity information received from the configured OIDC provider before using that information to identify, create, link, or authorize an Immich user.

SR-OIDC-04 — Account Linking:
The Immich authentication system shall link an external OIDC identity to a local account only after successful validation and shall prevent an existing OAuth association from being silently replaced.

SR-OIDC-05 — Account Provisioning:
The Immich authentication system shall provide administrators with the ability to disable automatic creation of local accounts from externally authenticated identities.

SR-OIDC-06 — Role Claims:
The Immich authentication system shall accept authorization-role claims only from a successfully validated OIDC identity and shall restrict accepted values to recognized Immich roles.

SR-OIDC-07 — OIDC TLS:
The Immich authentication system shall validate TLS certificates for OIDC discovery, token, user-info, and JWKS communications by default.

SR-OIDC-08 — OAuth Resource SSRF:
The Immich server shall validate externally supplied OAuth profile-resource URLs before initiating server-side network requests and shall prevent those URLs from accessing prohibited internal resources. 

#### Immich Implementation Evidence

The attached analysis concludes that current Immich provides direct implementation support for several of these requirements.
For SR-OIDC-01, AuthService.callback() obtains the expected OAuth state and rejects the request with "OAuth state is missing" when no state is available. The expected state is then supplied to OAuth processing. The document therefore assesses state validation as Supported. 
…
For SR-OIDC-02, the callback retrieves the PKCE code verifier and explicitly rejects authentication when it is absent with "OAuth code verifier is missing". The assessment is Supported in the current design.

For SR-OIDC-03, Immich uses OIDC processing and supports configuration of the ID-token signing algorithm. The document also records a historical TLS-validation weakness and states that v3.0.0 corrected that issue. The assessment is therefore Supported in current patched releases, historically deficient.

For SR-OIDC-04, the current authentication logic checks whether an email-matched account already has an OAuth identity association and rejects authentication rather than silently replacing that association. The document assesses this as Supported, with dependency on trusted IdP configuration and claim integrity. 

For SR-OIDC-05, Immich provides an Auto Register OAuth setting. When auto-registration is disabled and no user exists, authentication fails rather than automatically provisioning the account. This is assessed as Supported. 

For SR-OIDC-06, Immich supports role claims and can use the configured role claim to determine administrative status. This is technically supported, but the IdP and its claim governance become security-critical. 
For SR-OIDC-07, the document records a historical OIDC TLS-validation vulnerability involving discovery, token, user-info, and JWKS requests and states that v3.0.0 corrected it. The assessment is current releases: supported; affected older releases: not supported.

For SR-OIDC-08, the document records an OAuth profile-picture SSRF vulnerability and identifies v3.0.0 as the patched release. The assessment is historically inadequate; patched in v3.0.0.


## 3. Consolidated Security Requirements

| ID | Security Requirement | Derived From | Security Property / Control | Immich Support | Evidence |
|---|---|---|---|---|---|
| **SR-AUTH-01** | The system shall authenticate a user before establishing an authenticated Immich session and granting access to protected resources. | Authentication/Login | Authentication & Session Management | **Supported** | The OAuth flow establishes the Immich session only after external authentication, callback processing, identity validation, and account association. |
| **SR-ASSET-01** | The system shall authorize access to private assets according to the authenticated user's ownership or granted permissions before permitting protected asset operations. | Private Asset Access | Object-Level Authorization | * | Requires evidence from the team's Private Asset Access analysis. |
| **SR-SHARE-01** | Shared-link credentials shall grant access only to the resources and operations explicitly authorized by the share. | Shared Links | Scoped Authorization / Least Privilege  Requires evidence from the team's Shared Links analysis. |
| **SR-ADMIN-01** | Administrative functionality shall be available only to users whose administrator privileges have been successfully established and authorized. | Administrative Access | RBAC / Least Privilege | **Partially addressed** | OAuth role claims can affect local administrative status; only trusted, recognized role claims should affect privileges. |
| **SR-OIDC-01** | The system shall protect external authentication by validating OAuth state, requiring PKCE, cryptographically validating OIDC identity, securely linking accounts, controlling automatic registration and role claims, validating IdP TLS, and validating externally supplied profile resources. | OAuth/OIDC | Federated Authentication / Transaction Integrity | **Supported in current patched design, with IdP/configuration dependencies** | The document identifies direct state and PKCE checks plus controls for identity validation, account linking, registration, roles, TLS, and SSRF. |
The following table consolidates the functional security requirements derived from the five misuse-case analyses and maps them to the threats and security controls that motivated them.

## 4. AI-Assisted Use/Misuse Case Analysis

### 4.1 Prompt Used

You are an expert software security requirement engineer. Your job is to suggest misuse cases for a particular description of a use case diagram. Misuse cases need to be introduced in stages as back-and-forth analysis by introducing security countermeasures in response to a misuse case. For the Immich OAuth/OIDC External Authentication use case, identify the actors/interactions, relevant contextualized misusers, misuse cases, countermeasures, and derived security requirements. Then evaluate Immich's documentation and code to determine how well the implementation supports those requirements.

### 4.2 Suggestions Produced

The analysis produced eight principal misuse cases:
1. OAuth callback/CSRF manipulation.
2. Authorization-code interception.
3. Forged or manipulated OIDC identity.
4. Account-linking/account-takeover attacks.
5. Unauthorized automatic account creation.
6. Privilege escalation through role claims.
7. MITM attacks against Immich-to-IdP communications.
8. SSRF through OAuth profile resources.
Corresponding countermeasures included state validation, PKCE, cryptographic OIDC identity validation, controlled account linking, configurable auto-registration, trusted role claims, TLS certificate validation, and URL/network destination validation

### 4.3 Improvements Made to the Diagrams

The AI-assisted analysis expanded the initial use-case diagram from page 1 by introducing explicit malicious actors and <<threatens>> relationships. It also added security use cases and <<mitigates>> relationships.
The most significant improvement is that security controls are now traceable:
Misuser → Misuse Case → Threatened Use Case → Countermeasure → Security Requirement
For example:
Network Attacker → Impersonate IdP → Threatens OIDC Authentication → TLS Certificate Validation → SR-OIDC-07
This makes the diagram useful for requirements engineering rather than merely illustrating the normal authentication workflow.

### 4.4 Usefulness and Limitations

AI assistance was useful for systematically exploring how an attacker could adapt after each security control was introduced. It helped transform a normal OAuth/OIDC workflow into explicit security requirements.
The main limitation is that suggested misuse cases do not prove that Immich contains a vulnerability or implements a countermeasure. Implementation claims therefore require verification against documentation, source code, tests, and security advisories. The attached document follows that distinction by separately evaluating whether each derived requirement is actually supported.

---

## 5. Alignment with Immich Security Features

### 5.1 Supported Security Requirements

Supported Security Requirements
According to the attached analysis, current Immich supports:
- OAuth state validation.
- PKCE code-verifier handling.
- OIDC identity processing.
- Configurable automatic OAuth registration.
- Protection against silently replacing an already-linked OAuth identity.
- OAuth role-claim processing.
- TLS protection in patched releases.
- Protection against the documented OAuth profile-resource SSRF issue in patched releases.
The overall assessment also identifies back-channel logout support and states that Immich documentation expects the Authorization Code flow

### 5.2 Partially Supported Security Requirements

Some security requirements depend on the external IdP rather than Immich alone.
Role-based authorization is a clear example. Immich can consume a configured role claim and use it to establish administrative status, but the correctness of that decision depends on the IdP correctly authenticating the user and securely governing the role claim

### 5.3 Identified Security Gaps

The document identifies important historical gaps rather than claiming they necessarily remain in a current patched installation.
The first was insufficient TLS certificate verification for OIDC discovery, token, user-info, and JWKS communications. The document identifies v3.0.0 as the patched release. 
The second involved SSRF through OAuth profile-picture synchronization, where an externally supplied URL could cause Immich to access internal resources. The document likewise identifies v3.0.0 as patched.

### 5.4 Sufficiency of Existing Security Features

The analyzed controls provide meaningful protection for OAuth/OIDC authentication when Immich is current and correctly configured. State validation protects the authentication transaction, PKCE protects the authorization-code exchange, OIDC validation establishes identity trust, controlled linking protects local accounts, and TLS protects the Immich-to-IdP trust boundary.
These controls are not sufficient in isolation. Security also depends on a trusted and correctly configured IdP, appropriate role-claim governance, secure automatic-registration settings, and use of a patched Immich version. The historical TLS and SSRF findings demonstrate why implementation verification remains necessary even when the architecture contains appropriate security mechanisms. 

## 6. Security Configuration and Installation Documentation Review

### 6.1 Existing Security Guidance

The attached document identifies Immich's OAuth/OIDC documentation and authentication-service source as the principal sources for configuring and understanding external authentication. It also indicates that Immich documents OIDC support and expects an Authorization Code flow. 
The analysis identifies configurable security-relevant features including:
- External/OIDC authentication.
- Automatic OAuth registration.
- OIDC identity processing.
- OAuth role claims.
- PKCE.
- OAuth state handling.
- Account linking.
- Back-channel logout.

### 6.2 Missing or Unclear Security Guidance

Based strictly on the attached analysis, several areas would benefit from clearer security-oriented documentation.
First, the security consequences of automatic OAuth registration should be emphasized because enabling it determines whether any successfully authenticated external identity may obtain a local Immich account.
Second, the documentation should emphasize that role claims form an authorization trust boundary. A compromised or incorrectly configured IdP role mapping can propagate administrative privileges into Immich.
Third, administrators should be clearly informed that MFA and rate limiting are delegated to the OAuth server and therefore must be configured there when required.
Finally, OIDC documentation should prominently explain the security significance of TLS certificate validation and external profile-resource handling because both correspond to historical vulnerabilities identified by the analysis.

### 6.3 Recommended Documentation Improvements

Immich's OAuth/OIDC documentation should include a dedicated Security Considerations section containing a concise deployment checklist:
1. Use a current patched Immich release.
2. Configure only trusted OIDC issuers.
3. Require HTTPS and valid TLS certificates for IdP communications.
4. Protect OAuth client secrets.
5. Configure MFA and authentication rate limiting at the IdP where required.
6. Review whether Auto Register OAuth should be enabled.
7. Restrict role claims to controlled, recognized values.
8. Treat administrator-role mappings as security-critical configuration.
9. Review account-linking behavior before enabling external authentication for existing local users.
10. Validate redirect/callback configuration.
11. Avoid unnecessary identity claims and scopes.
12. Keep SSRF protections enabled for externally supplied profile resources.
13. Test OAuth login, account linking, role mapping, and logout after authentication configuration changes.
14. Review relevant Immich security advisories before deployment and upgrades.
This recommendation follows directly from the document's central finding: Immich's current OAuth/OIDC architecture contains meaningful controls, but the security of the authentication boundary also depends heavily on IdP configuration, claim integrity, deployment configuration, and keeping Immich patched. 

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

The five essential interactions were divided among team members so that each person was responsible for developing one complete use/misuse case analysis. Each analysis included identifying actors and interactions, developing misuse cases and security countermeasures, deriving security requirements, and comparing those requirements with Immich's documentation or source code.

GitHub Issues and the Project Board were used to assign and track work. Team members worked on separate branches and used pull requests to integrate completed work into the main report. Pull requests also provided an opportunity for teammates to review contributions and suggest changes before they were merged.

In addition to the five individual analyses, report-wide responsibilities were distributed across the team. Adu was assigned the security documentation review, Marc assisted with diagram consistency and notation, Dillon compiled the consolidated security requirements, Matthew assisted with verifying requirements against Immich documentation and source code, and Christian maintained the Project Board and integrated the final Markdown report.

The team also communicated outside GitHub to coordinate progress, resolve repository-access issues, identify work that still needed to be integrated, and avoid overlapping edits as the final report was assembled. Before submission, the completed analyses and report-wide sections were brought together for a final team review.

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

I was tasked with the analysis of uploading assets and private asset access section. I looked into how our Photography Studio Member uploads client shoots to Immich and views their private assets, and what might happen if another user tried to access those assets without permission. During my analysis I looked at two types of misusers, a curious studio employee with a valid Immich account and someone who acquired a staff member's lost/stolen device with an active logged-in session on it.

One thing that I learned while working on my part was that the biggest threat or risk isn't an outside breaking in but an authenticated user accessing something they shouldn't. The employee already passes authentication into the server, so the asset protection must come from ownership checks instead of the login page. While working through the other misuser, my analysis had to go even further because the person with a stolen device has access to an authenticated active session and will pass an ownership check too, which led to the features of session revocation, the Locked Folder, and PIN-elevation as a second layer of security inside of an authenticated account.

### Dillon

For this assignment, I was responsible for the Shared Albums and Public Links section. I focused on how a photography studio could use Immich to share photos with clients and what security problems could come from public shared links. I created the use/misuse case diagram and looked at two main risks, someone getting a shared link that was not intended for them and someone trying to use a link after it had expired.

One thing I learned from this part of the project was how normal features can create security risks depending on how they are used. Public links are useful for easily sharing photos with clients, but they can also be forwarded or exposed to other people. Adding password protection and expiration dates helped show how security controls can be connected directly to specific misuse cases instead of just listing general security features.

I also reviewed Immich's documentation and source code to compare the security requirements from my analysis with what Immich actually supports. This showed me that the password and expiration protections from my diagram are already implemented in Immich. I am also helping combine the security requirements from each team member into the final set of requirements for the project.

### 8.1 Combined Team Reflection

TBD

---

## References

- Immich. "Access Utilities." *Immich GitHub Repository*.
  https://github.com/immich-app/immich/blob/main/server/src/utils/access.ts

- Immich. "Access Repository." *Immich GitHub Repository*.
  https://github.com/immich-app/immich/blob/main/server/src/repositories/access.repository.ts

- Immich. "Release v3.1.0." *Immich GitHub Repository*.
  https://github.com/immich-app/immich/releases/tag/v3.1.0

- Immich. "Pull Request #31735: Document Locked Folder Session Behaviour." *Immich GitHub Repository*.
  https://github.com/immich-app/immich/pull/31735

- Immich. "Auth Service." *Immich GitHub Repository*.  
  https://github.com/immich-app/immich/blob/main/server/src/services/auth.service.ts

- Immich. "OAuth." *Immich Documentation*.  
  https://docs.immich.app/administration/oauth/

- Immich. "System Settings." *Immich Documentation*.  
  https://docs.immich.app/administration/system-settings/
  
- Immich. "Sharing." *Immich Documentation*.  
  https://docs.immich.app/features/sharing/

- Immich. "Shared Link Service." *Immich GitHub Repository*.  
  https://github.com/immich-app/immich/blob/main/server/src/services/shared-link.service.ts
  
