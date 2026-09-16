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

*Immich license, contribution procedures, and contributor agreements to be added.*

## Security-Related History

*Summary of Immich's security-related history, known vulnerabilities, security decisions, and security feature changes to be added.*

## Team Reflection

### Individual Reflections

**Christian Theisen**

*Reflection to be added.*

**Matthew Rayl**

While working on this proposal, I learned quite a bit about what the architecture looks like for Immich and how different smaller services/components work together to make up a sophisticated photo management program. The thing I found most useful was reading through Immich’s documentation accessible on their github. Immich has extremely detailed diagrams on the architecture of their software which helped out a lot while working on the operational environment.

**Team Member 3**

*Reflection to be added.*

**Team Member 4**

*Reflection to be added.*

### Team Reflection

*Combined team reflection to be added.*
