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

*Threat analysis to be added.*

### Security Features

*Security features to be added.*

## Team Motivation

Our team selected Immich because it is an actively developed open-source application with clear security requirements that relate to a realistic use case. Immich stores and manages personal photos and videos, which users would reasonably expect to remain private unless they intentionally choose to share them. Features such as multiple user accounts, shared albums, public sharing, OAuth authentication, and administrative user management make authentication and authorization important parts of the application's security.

Immich also has an active open-source community and continues to receive frequent updates and community contributions. This gives our team an opportunity to apply software assurance techniques to a real-world project while examining software that has meaningful security and privacy expectations.

## Open-Source Project Description

Immich is a self-hosted photo and video management application designed to give users control over storing, organizing, backing up, and sharing their personal media. It provides web and mobile applications and includes features such as automatic mobile backups, multiple user accounts, shared albums, public sharing, partner sharing, facial recognition and search, OAuth authentication, API keys, and administrative user management.

Immich has a large and active open-source community. Its GitHub repository has over 114,000 stars, approximately 6,900 forks, and hundreds of open issues and pull requests. Development remains active, with frequent commits and releases as well as contributions from the community. At the time of this proposal, Immich continues to publish frequent releases and maintains issues specifically identified as appropriate for first-time contributors. This level of activity was an important consideration for our team because it provides an active project where we can observe development practices and potentially interact with contributors throughout the semester.

The Immich ecosystem contains web, server, mobile, and machine-learning components and uses several technologies and programming languages. Major technologies include TypeScript, Svelte/SvelteKit, NestJS, PostgreSQL, and Python. Immich provides documentation covering installation, configuration, administration, development, and contribution procedures. For self-hosted production environments, Docker Compose is the recommended deployment method.

**Sources:** [Immich GitHub Repository](https://github.com/immich-app/immich) | [Immich Releases](https://github.com/immich-app/immich/releases) | [Immich Contribution Opportunities](https://github.com/immich-app/immich/contribute) | [Immich Documentation](https://docs.immich.app/)

## License and Contribution Process

*Immich license, contribution procedures, and contributor agreements to be added.*

## Security-Related History

*Summary of Immich's security-related history, known vulnerabilities, security decisions, and security feature changes to be added.*

## Team Reflection

### Individual Reflections

**Christian Theisen**

While working on this proposal, I learned more about how GitHub issues and project boards can be used to organize and track work within a team. I was already familiar with branches and commits, but this was my first time using GitHub issues as part of a development workflow and connecting an issue to the work being completed on a branch. I also learned more about Immich as an open-source project and how active its development community is. Researching the project helped me better understand how authentication and authorization can be evaluated from a software assurance perspective by defining security requirements and gathering evidence to determine whether those requirements are being met.

**Team Member 2**

*Reflection to be added.*

**Team Member 3**

*Reflection to be added.*

**Team Member 4**

*Reflection to be added.*

### Team Reflection

*Combined team reflection to be added.*
