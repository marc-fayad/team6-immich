# Team 6 Project Proposal – Immich

## Team Repository

Our team's public GitHub repository and collaborative workspace can be found here:

https://github.com/marc-fayad/team6-immich

## Selected Open-Source Project

**Project:** Immich

**Open-Source Repository:** https://github.com/immich-app/immich

## Project Scope

Our team will focus on assessing Immich's authentication and authorization mechanisms, particularly whether access controls adequately protect private and shared photo and video assets in a self-hosted environment.

## Hypothetical Operational Environment

Our Operational Environment is a small photography studio business with five staff members that hosts Immich on a Linux server using Docker Compose. Staff will use web and mobile apps for Immich to organize and back up client photos, keep unfinished work private, and deliver completed work to clients via shared links. The Studio’s server hosts a couple major components:

**immich-server:** The server is what handles REST API requests from the web browser and mobile apps, performs authorization checks and takes care of background jobs like metadata extraction/thumbnail generation.

**immich-machine-learning:** Python service that handles facial recognition and smart search. 

**PostgreSQL:** Database that stores users, authorization info, photo albums, etc.

**Redis:** job queue for background processing

**Local File Storage:** Original photos, videos, and thumbnails.

A Reverse Proxy with TLS is the only component used that is exposed to the internet via port 443 under a studio-owned domain. The traffic is then forwarded to the Immich service on the internal network. PostgreSQL, Redis, and the Immich ML service are not published outside of the Docker Network. 

The Studio staff expect Immich to maintain confidentiality for each user’s private photos unless they are deliberately shared, and to limit shared content to only the intended people. 

### Systems Engineering Diagram

*Diagram and description to be added.*

### Perceived Threats

**Unauthorized Access to Private Assets:** An authorized user viewing, downloading, modifying, or deleting another user's photos

**Privilege Escalation:** A regular user gaining access to admin functions or an album viewer accessing editor functions

**Shared-Link Abuse:** Links being forwarded to unintended people, left active without an expiration date, or used to reach photos outside of the link's scope

**Credential and Token Theft:** Compromised passwords, stolen session tokens or leaked API keys

**Authentication Misconfiguration:** Weak or incorrect OAuth/OpenID Connect setup allowing account creation/takeover

**Infrastructure Exposure:** Misconfigured database that gives direct access to unencrypted assets, bypassing the application

### Security Features

- Proper local password login and OpenID Connect single sign-on
- Session management and per-device revoation
- API keys with scoped permissions
- Separate admin and user roles as well as per-user storage limits
- Owner-based access checks on all assets/photo albums
- Album sharing with editor and viewer roles
- Shared links with optional passwords, expiration dates, download controls, metadata hiding
- PIN-Protected locked folders for confidential assets
- Proper internal network for backing services like PostgreSQL and Redis

## Team Motivation

*Team motivation for selecting Immich to be added.*

## Open-Source Project Description

*Description of Immich, contributors, activity, usage, popularity, languages, platforms, documentation, and other project information to be added.*

## License and Contribution Process

Immich is licensed under the GNU Affero General Public License v3.0 (AGPL-3.0). This license allows users to use, modify, and distribute the software, while requiring modified source code to remain available under certain conditions, including when a modified version is made available to users over a network. Immich switched from the MIT License to AGPLv3 in 2024 to help ensure that modifications to the project remain available to the open-source community.

Immich's contribution guidelines ask contributors to keep pull requests focused on a single change and to discuss larger or more impactful changes with the maintainers before beginning development. The project provides development setup documentation and identifies good-first-issue items for new contributors. The current contribution guidelines do not identify a separate Contributor License Agreement (CLA) or Developer Certificate of Origin (DCO) that contributors are required to sign.

Sources: Immich License https://github.com/immich-app/immich/blob/main/LICENSE | Contribution Guidelines https://github.com/immich-app/immich/blob/main/CONTRIBUTING.md | Developer Setup https://docs.immich.app/developer/setup/

## Security-Related History

Immich's security history shows why authentication and authorization are important for protecting private and shared photo assets. In 2026, Immich disclosed an authorization vulnerability involving shared links. Someone with access to a shared-link key could potentially add other assets belonging to the owner to the shared link, exposing private content that the owner did not originally intend to share.

Another vulnerability disclosed in 2026 involved password-protected shared albums. The shared-album password could be included in URL query parameters, potentially exposing it through browser history, server or proxy logs, or referrer information. An attacker who obtained the password could potentially use it to access the shared media. These vulnerabilities directly relate to our project's focus because they demonstrate how failures in authentication and authorization controls can result in unintended access to private or shared assets.

Sources: Shared-Link Authorization Bypass Advisory https://github.com/immich-app/immich/security/advisories/GHSA-hvq7-hq9r-8gjr | Shared-Link Password Disclosure Advisory https://github.com/immich-app/immich/security/advisories/GHSA-78x4-6x83-jx75 | Immich Security Advisories https://github.com/immich-app/immich/security/advisories

## Team Reflection

### Individual Reflections

**Christian Theisen**

*Reflection to be added.*

**Matthew Rayl**

While working on this proposal, I learned quite a bit about what the architecture looks like for Immich and how different smaller services/components work together to make up a sophisticated photo management program. The thing I found most useful was reading through Immich’s documentation accessible on their github. Immich has extremely detailed diagrams on the architecture of their software which helped out a lot while working on the operational environment.

**Dillon Haliburton**

This assignment helped me understand more about how Immich handles security when users are sharing private photos and videos. I thought the most interesting part was looking into some of Immich's past security issues involving shared links and access controls. Seeing actual examples of how a mistake in authorization could expose photos that were supposed to stay private helped me understand why these security features are so important. I also learned more about how open-source projects handle licensing and contributions from people outside of the main development team.

**Team Member 4**

*Reflection to be added.*

### Team Reflection

*Combined team reflection to be added.*
