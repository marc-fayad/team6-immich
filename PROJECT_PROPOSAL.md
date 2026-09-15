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

*Threat analysis to be added.*

### Security Features

*Security features to be added.*

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

**Team Member 2**

*Reflection to be added.*

**Team Member 3**

*Reflection to be added.*

**Team Member 4**

*Reflection to be added.*

### Team Reflection

*Combined team reflection to be added.*
