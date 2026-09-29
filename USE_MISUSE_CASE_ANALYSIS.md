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

*Diagram to be added.*

#### Derived Security Requirements

The iterative analysis produced these security requirements for Upload and Private Asset Access:

**SR-ASSET-01 - Ownership Assignment:** 

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

### Christian

TBD

### Adu

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

TBD
