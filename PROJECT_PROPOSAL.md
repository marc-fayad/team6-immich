# Team 6 Project Proposal – Immich

## Team Repository

Our team's public GitHub repository and collaborative workspace can be found here:

https://github.com/marc-fayad/team6-immich

## Selected Open-Source Project

**Project:** Immich

**Open-Source Repository:** https://github.com/immich-app/immich

## Project Scope

Team 6 will focus on assessing Immich's authentication and authorization mechanisms, particularly whether access controls adequately protect private and shared photo and video assets in a self-hosted environment.

Rather than attempting to assess the security of the entire Immich application, our analysis will focus on a limited set of security-related functionality involving user authentication, access to user-owned assets, and the sharing of assets between users or through public sharing features. Throughout the semester, we will develop security requirements for these features and collect evidence to determine whether the software satisfies those requirements.

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

Our team selected Immich because it is an actively developed open-source application with clear security requirements that relate to a realistic use case. Immich stores and manages personal photos and videos, which users would reasonably expect to remain private unless they intentionally choose to share them. Features such as multiple user accounts, shared albums, public sharing, OAuth authentication, and administrative user management make authentication and authorization important parts of the application's security.

Immich also has an active open-source community and continues to receive frequent updates and community contributions. This gives our team an opportunity to apply software assurance techniques to a real-world project while examining software that has meaningful security and privacy expectations.

## Open-Source Project Description

Immich is a self-hosted photo and video management application designed to give users control over storing, organizing, backing up, and sharing their personal media. It provides web and mobile applications and includes features such as automatic mobile backups, multiple user accounts, shared albums, public sharing, partner sharing, facial recognition and search, OAuth authentication, API keys, and administrative user management.

Immich has a large and active open-source community. Its GitHub repository has over 114,000 stars, approximately 6,900 forks, and hundreds of open issues and pull requests. Development remains active, with frequent commits and releases as well as contributions from the community. At the time of this proposal, Immich continues to publish frequent releases and maintains issues specifically identified as appropriate for first-time contributors. This level of activity was an important consideration for our team because it provides an active project where we can observe development practices and potentially interact with contributors throughout the semester.

The Immich ecosystem contains web, server, mobile, and machine-learning components and uses several technologies and programming languages. Major technologies include TypeScript, Svelte/SvelteKit, NestJS, PostgreSQL, and Python. Immich provides documentation covering installation, configuration, administration, development, and contribution procedures. For self-hosted production environments, Docker Compose is the recommended deployment method.

**Sources:** [Immich GitHub Repository](https://github.com/immich-app/immich) | [Immich Releases](https://github.com/immich-app/immich/releases) | [Immich Contribution Opportunities](https://github.com/immich-app/immich/contribute) | [Immich Documentation](https://docs.immich.app/)

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

While working on this proposal, I learned more about how GitHub issues and project boards can be used to organize and track work within a team. I was already familiar with branches and commits, but this was my first time using GitHub issues as part of a development workflow and connecting an issue to the work being completed on a branch. I also learned more about Immich as an open-source project and how active its development community is. Researching the project helped me better understand how authentication and authorization can be evaluated from a software assurance perspective by defining security requirements and gathering evidence to determine whether those requirements are being met.

**Matthew Rayl**

While working on this proposal, I learned quite a bit about what the architecture looks like for Immich and how different smaller services/components work together to make up a sophisticated photo management program. The thing I found most useful was reading through Immich’s documentation accessible on their github. Immich has extremely detailed diagrams on the architecture of their software which helped out a lot while working on the operational environment.

**Dillon Haliburton**

This assignment helped me understand more about how Immich handles security when users are sharing private photos and videos. I thought the most interesting part was looking into some of Immich's past security issues involving shared links and access controls. Seeing actual examples of how a mistake in authorization could expose photos that were supposed to stay private helped me understand why these security features are so important. I also learned more about how open-source projects handle licensing and contributions from people outside of the main development team.

**Team Member 4**

*Reflection to be added.*

### Team Reflection

*Combined team reflection to be added.*
