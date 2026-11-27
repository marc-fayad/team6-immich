# Team 6 — Immich Security Code Review

**Course:** Software Assurance  
**Project:** Immich  
**Assignment due:** December 7, 2026  
**Status:** Working template — findings pending verification

## Part 1: Code Review

### 1. Code Review Strategy

#### 1.1 Objectives and Methodology

Our team will use a combination of **scenario/weakness-based review** and a **CWE checklist-based review**. We will focus on security-sensitive Immich code associated with our prior misuse cases, assurance claims, and threat models rather than attempt to inspect the entire codebase. We will supplement manual review with appropriate automated code-scanning tools where available.

#### 1.2 Scope and Team Responsibilities

| Reviewer | Security area | Relevant files/functions | Related prior artifact |
|---|---|---|---|
| Christian | Authentication and login | To identify | Authentication misuse case |
| Matthew | Asset upload and private access | To identify | Asset access misuse case |
| Dillon | Shared albums and public links | To identify | Sharing misuse case |
| Marc | Administrative access and user management | To identify | Administrative misuse case |
| Adu | OAuth/OIDC and external authentication | To identify | External authentication misuse case |

Record the Immich repository version/commit reviewed: **TBD**.

#### 1.3 Priority CWE Checklist

The following are **candidate review targets, not confirmed Immich vulnerabilities**. Finalize a focused list of 5–10 CWEs as a team and justify each selection based on the source code and architecture.

| CWE | Weakness | Review relevance |
|---|---|---|
| CWE-287 | Improper Authentication | Login and external authentication |
| CWE-307 | Improper Restriction of Excessive Authentication Attempts | Login abuse |
| CWE-862 | Missing Authorization | Private assets and administrative operations |
| CWE-863 | Incorrect Authorization | Resource-level access decisions |
| CWE-639 | Authorization Bypass Through User-Controlled Key | Asset and sharing identifiers |
| CWE-269 | Improper Privilege Management | Administrative functions |
| CWE-200 | Exposure of Sensitive Information to an Unauthorized Actor | Private assets and sharing |
| CWE-601 | URL Redirection to Untrusted Site | External authentication redirects, if applicable |

#### 1.4 Automated Scanning Strategy

- **Tool(s):** TBD after confirming compatibility with Immich's languages and repository structure.
- **Scope/configuration:** TBD.
- **Scan date and reviewed commit:** TBD.
- **Output links:** TBD.
- **Validation:** Manually triage significant alerts and distinguish true positives, false positives, and unverified warnings.

### 2. Anticipated Challenges and Mitigations

#### 2.1 Challenges Expected Before Review

- Large codebase and limited review time.
- Security logic distributed across controllers, services, and shared authorization utilities.
- Difficulty distinguishing a potential weakness from an exploitable vulnerability.
- Scanner noise and tool limitations.
- Consistent evidence collection across five reviewers.

#### 2.2 How the Strategy Addresses These Challenges

- Limit each review to the code paths implicated by previous security analyses.
- Use a shared CWE checklist and finding format.
- Record source-code permalinks pinned to a commit, with relevant function names and line references.
- Verify scanner alerts against actual application behavior and controls.
- Use GitHub issues and pull requests to track ownership and peer review.

### 3. Manual Code Review Findings

For each reviewer, document **finding ID; reviewer; related misuse case, assurance claim, or threat model; reviewed commit; file/function/line link; review method; observed evidence; applicable CWE; potential security impact; recommended mitigation; and verification status**. If no weakness is found, explain what was examined and the limitations of the review.

#### 3.1 Authentication and Login — Christian

**Reviewed code paths:** TBD  
**Findings/evidence:** Pending review.

#### 3.2 Asset Upload and Private Access — Matthew

**Reviewed code paths:** TBD  
**Findings/evidence:** Pending review.

#### 3.3 Shared Albums and Public Links — Dillon

**Reviewed code paths:** TBD  
**Findings/evidence:** Pending review.

#### 3.4 Administrative Access and User Management — Marc

**Reviewed code paths:** TBD  
**Findings/evidence:** Pending review.

#### 3.5 OAuth/OIDC and External Authentication — Adu

**Reviewed code paths:** TBD  
Code path	                    | Security-sensitive functionality
----------------------------  | ---------------------------------------
https://github.com/immich-app/immich/blob/main/server/src/services/auth.service.ts? | OAuth callback processing, account linking, identity mapping and session management.
https://github.com/immich-app/immich/blob/main/server/src/repositories/oauth.repository.ts?| OAuth provider discovery, authorization code exchange, PKCE, token validation and profile retrieval.
https://github.com/immich-app/immich/blob/main/server/src/controllers/oauth.controller.ts?| OAuth API endpoints, callback requests and cookie management.
https://github.com/immich-app/immich/blob/main/server/src/dtos/auth.dto.ts? | Authentication request validation and OAuth input parameters.
https://github.com/immich-app/immich/blob/main/server/src/middleware/auth.guard.ts? | Authentication enforcement and protection of restricted endpoints.
https://github.com/immich-app/immich/blob/main/server/src/repositories/user.repository.ts? | External identity mapping and user account persistence
https://github.com/immich-app/immich/blob/main/server/src/dtos/config.dto.ts? | OAuth provider configuration and administrative security settings.
https://github.com/immich-app/immich/blob/main/mobile/lib/services/oauth.service.dart? | Mobile OAuth authentication and callback handling


**Findings/evidence:** Pending review.

1.Findings : OAuth state validationAccount takeover: CWE-352  Cross Site Request Forgery

Evidence: auth.service.ts, oauth.repository.ts, oauth.controller.ts.

Immich contains measures to validate OAuth state. However, the correct functioning of this protection is reliant on the secure binding of a state value to a genuine login process. A security advisory was issued upstream on November 10 2026 and identifies an OAuth account binding issue which can lead to account takeover.- Advisory: GHSA-7j23-rprg-3g5q- Vulnerable versions: v3.3.6 and earlier- Fixed in: v3.4.0- Upstream risk rating: High- Verified: Yes, as identified in the upstream advisory.

Recommendation: Upgrade affected deployments and introduce regression tests covering transaction binding, unauthorized account linking, state expiration and replay prevention.

2.Findings: Conditional PKCE usage

CWE: CWE-287 – Incorrect authentication

Evidence: oauth.repository.ts – authorize() and getProfileAndOAuthSid().

PKCE is conditionally used based on what the identity provider claims that it can do. Therefore, defence against interception of an authorisation code also relies, in part, on how the identity provider is set up. 

Recommendation: Enforce the use of identity providers that can utilise PKCE (SHA-256) and detail the security consequences of using those that cannot.

Verification: Behaviour has been seen. Exploitability unknown.

3.Finding :Email-driven auto-linking of accounts CWE: CWE-345 - Insufficient Verification of Data Authenticity

 Evidence: auth.service.ts - method: callback() In the Auth Service, external account providers can be linked to current users via their email address. The security of this operation relies upon the external party having correctly authenticated the email. 

Suggestion: Confirm that the user has control over the relevant email address prior to auto-linking. May wish to force explicit action by the account holder to link accounts. 

Assessment: Further checks needed. 

4.Finding : SSRF via images returned from 3rd-party profiles CWE: CWE-918 - Server-Side Request Forgery 

Evidence: auth.service.ts - syncProfilePicture(); oauth.repository.ts - getProfilePicture() As the code makes outgoing requests based on data received from external sources (i.e. social media profiles), there is a possibility of SSRF if the allowed targets are not tightly controlled. 

.Suggestion: Confirm controls in place around: private IPs, localhost, redirect chains, DNS and supported protocols. Assessment: Not confirmed as exploitable.

 5.Finding : Debug logging may record PII CWE: CWE-532 - Insertion of Sensitive Information into Log File 

Evidence: auth.service.ts - method: callback() OAuth profile details are logged during the auth flow at debug level. Given the variability between different OAuth providers and configurable logging settings, some form of PII could be logged. 

Suggestion: Only log key fields relating to the authentication process, and remove any further sensitive data from the OAuth profile. Assessment: Requires assessment of logged output. 

Finding 6: External roles affect admin rights CWE: CWE-269 - Improper Privilege Management 

Evidence: auth.service.ts - methods: callback(), getRoleClaim() Roles defined in the external OAuth system will map to the local system's admin-level permissions. Misconfiguration of the OAuth connection and lack of hardening of the corresponding claims would allow unauthorized elevation of access. 

Suggestion: Limit which external role(s) are used to determine elevated local permissions, and validate ability to elevate and de-elevate access. Assessment: Dependent on configuration of OAuth.


### 4. Automated Code Scanning Findings

| Tool/version | Scan scope and configuration | Output link| Findings and triage status |
|---|---|---|---|
| Semgrep version pending execution | OAuth-related TypeScript files; security and OWASP rules | evidence/semgrep-oauth.json | Pending execution |
| GitHub CodeQL version pending execution| JavaScript/TypeScript security and data-flow analysis |evidence/codeql-results.sarif | Pending execution|
| npm Audit version pending execution| Server-side npm dependencies and lockfile | evidence/npm-audit.json | Pending execution|
| Gitleaks version pending execution |Source repository and Git history for exposed credentials | evidence/gitleaks-report.json| Pending execution |
| Manual security review - November 26, 2026| Authentication service, OAuth repository and relevant upstream advisory | https://github.com/immich-app/immich/security/advisories/GHSA-7j23-rprg-3g5q)| Preliminary findings documented |


**Limitations and false positives:** TBD.

## Part 2: Key Findings and Contributions

### 5. Summary of Key Findings and Risk

Map significant findings from manual and/or automated review to CWEs. Evaluate risk in the hypothetical professional photography studio operating Immich.

| Finding ID | Security area | CWE | Supporting evidence | Likelihood | Impact | Overall risk | Verification status |
|---|---|---|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD | TBD | TBD | Pending |

**Risk assessment rationale:** TBD.  
**Distinction between potential threats and confirmed vulnerabilities:** TBD.

### 6. Planned or Ongoing Upstream Contributions

Describe possible documentation, design, code, or communication contributions to the upstream Immich project, including proposals informed by earlier class assignments. Do not claim a contribution was submitted unless it was.

**Proposed contribution(s):** TBD.  
**Upstream issue/PR/discussion links (if any):** TBD.

### 7. Team Collaboration and GitHub Task Tracking

**Team repository:** [Insert Team 6 GitHub repository URL]  
**Project board:** TBD  
**Assigned issues and pull requests:** TBD

### 8. Individual Reflections

Each member: What did you learn from the assignment? What did you find most useful?

#### Christian
Pending.

#### Matthew
Pending.

#### Dillon
Pending.

#### Marc
Pending.

#### Adu
Pending.

### 9. Combined Team Reflection

Synthesize the individual reflections into one cohesive team reflection. **Pending team input.**

### 10. References and Evidence

- Prior Team 6 misuse cases: TBD.
- Prior assurance cases: TBD.
- Prior threat models: TBD.
- Immich source repository and reviewed commit: TBD.
- CWE references: TBD.
- Scanner documentation and output links: TBD.
