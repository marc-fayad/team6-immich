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
**Findings/evidence:** Pending review.

### 4. Automated Code Scanning Findings

| Tool/version | Scan scope and configuration | Output link | Findings and triage status |
|---|---|---|---|
| TBD | TBD | TBD | Pending |

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
