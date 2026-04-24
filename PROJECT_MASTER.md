# PROJECT_MASTER

## Mission

Turn Zanjir into a publishable, presentation-ready base distribution for a federated, privacy-oriented, self-hosted Matrix deployment that works on domestic or internal-only networks with mixed operator skill levels.

This session does not treat Zanjir as a greenfield messaging app. It treats it as an existing Matrix packaging project that must be assessed before any implementation work.

## Current Repository Understanding Summary

Zanjir is primarily a deployment bundle and installer, not a custom messaging protocol or full custom application.

Current repository facts from inspected files:

- The active runtime stack is `Conduit + Element Web + Caddy + coturn + a small Flask admin panel` in [docker-compose.yml](/home/saeid/Documents/Codex/Zanjir/docker-compose.yml:1).
- The admin panel is custom but thin, and many admin actions are placeholders rather than complete server-management logic in [admin/app.py](/home/saeid/Documents/Codex/Zanjir/admin/app.py:121).
- The installer now supports explicit `isolated` versus `federated` mode selection, writes deployment metadata into `.env`, copies private-CA PEM material into `./certs` when selected, and activates the matching Caddy template in [install.sh](/home/saeid/Documents/Codex/Zanjir/install.sh:1).
- The repository has begun cleanup of stale Dendrite-era drift: dead Dendrite installer functions and helper files were removed, and the English README no longer points operators to Dendrite-only account creation and PostgreSQL-era backup guidance.
- Matrix federation is now env-controlled in the compose stack via `CONDUIT_ALLOW_FEDERATION: "${FEDERATION_ENABLED:-false}"` in [docker-compose.yml](/home/saeid/Documents/Codex/Zanjir/docker-compose.yml:17), but the installer still defaults new installs to isolated mode.

## What Zanjir Actually Is

Zanjir is best understood as:

- a Matrix homeserver packaging and onboarding project
- a single-node deployment kit
- a branding/configuration layer around upstream Matrix components
- a convenience installer for operators in constrained environments

It is not, today:

- a new federated protocol
- a distributed control plane
- a complete multi-server federation management product
- a mature admin product

## Current Status

Current repo state is a single-server Matrix deployment biased toward isolated use:

- federation disabled by default and described as intentionally disabled
- public-internet assumptions still present in parts of the config and docs
- IP-only mode exists, but it is designed for local access convenience, not robust multi-server federation
- Element compatibility already exists
- security and operational guidance are incomplete
- some documentation still needs deeper operational hardening work, but the main README, Persian README, and migration guides now describe the Conduit-based runtime and deployment modes more truthfully

## Target Product Vision

Target state for this project:

- self-hostable by families, communities, and trusted operators
- each server owns its local accounts and operations
- independently operated servers can federate fully and usefully over domestic/internal networks
- failure of one server does not collapse the whole network
- cross-server public rooms are usable
- private rooms and direct messages use Matrix E2EE where supported by clients
- Element is reused for the initial final-product path unless a hard blocker appears
- deployment, recovery, upgrades, and hardening are first-class concerns
- the result should be fit for public presentation and eventual publication, not just a lab proof

## Explicit Constraints

- Must work when international internet is unavailable.
- Must not assume public DNS, public CA reachability, or stable upstream package access.
- Must not overclaim anonymity or metadata protection.
- Must prefer configuration, deployment engineering, and documentation over custom code when they are sufficient.
- Must remain maintainable by small operators.

## Threat Model Summary

Primary concerns:

- hostile or compromised participating homeservers
- operator mistakes and weak defaults
- credential theft and secret leakage
- backup loss or unsafe restore
- misissued trust due to weak bootstrap
- traffic inspection by network observers on the domestic network
- abuse, spam, and unwanted invitations from federated peers

Non-goals:

- covert intrusion capabilities
- bypass of lawful or technical network boundaries
- global anonymity guarantees

## Trust Model Summary

- Each homeserver operator is trusted for its own infrastructure, backups, logs, and user lifecycle.
- Federated peers are only partially trusted.
- A participating homeserver can learn substantial metadata about traffic it handles.
- Clients must trust their own homeserver for a large amount of metadata and for key-delivery-related behavior typical of Matrix deployments.
- Trusted introducer servers may help bootstrap federation, but must not become mandatory permanent coordination points.

## Federation Model Summary

Answer to required question 1:

Yes. Zanjir already gets Matrix federation from its upstream homeserver stack if the homeserver is configured to allow federation and if transport, naming, TLS, and peer reachability are made to work. The repo currently disables that capability in active compose config rather than lacking the protocol stack itself.

Answer to required question 2:

What is still missing for the target product:

- turning federation on safely
- replacing public-DNS/public-CA assumptions with domestic-network naming and trust guidance
- defining bootstrap and discovery workflows
- documenting peer onboarding and trust boundaries
- hardening exposed services
- adding recovery, upgrade, moderation, and abuse procedures
- cleaning repo confusion caused by stale Dendrite content

## Bootstrap Model Summary

Chosen direction:

- minimally centralized bootstrap
- server joins happen through one of four supported paths:
- invite-based join from an existing participant
- known-entry-point operator list
- room-alias-based join where naming works
- operator-provided trusted introducer servers

No permanent global registry is assumed.

## Discovery Model Summary

Chosen direction:

- incremental federation learning
- discovery through room membership, invitations, and known peers
- optional trusted introducer list for first contact
- optional room directory or documented alias publication on introducer servers

This is practical and maintainable. Full network-wide decentralized discovery is not the target.

## Internal Naming Strategy Summary

Preferred order:

1. internal DNS with operator-managed domains if available
2. operator-managed local domain convention such as `hs1.matrix.local` or an organization-owned internal zone
3. static host mapping with well-documented peer address onboarding
4. direct peer address onboarding only as a fallback

Important caveat:

IP-only self-signed mode may be acceptable for local testing or tightly controlled small deployments, but it is a weak base for durable federation between independently operated servers. For full internal-network federation, prefer stable internal domains plus valid TLS certificates.

Certificate preference order:

1. public or internationally trusted CA certificates if the internal domains can actually be validated in the deployment environment
2. private/internal CA certificates if public CA issuance is not possible
3. static host mapping plus private/internal CA only if DNS fails

Plain HTTP is not a valid federation architecture fallback for the final product.

## Deployment Model Summary

Preferred deployment model for domestic/internal networks:

- one homeserver stack per operator
- containerized deployment retained initially
- stable internal naming per homeserver
- reverse proxy terminated with valid TLS for the chosen trust model
- explicit port and peer connectivity documentation
- no dependency on `matrix.org`, public identity servers, or international services for core messaging

## Hardening Principles Summary

- secure defaults before convenience
- disable unnecessary exposure
- minimize secrets in logs and UI
- document trust boundaries
- define backup and restore before rollout
- prefer auditable config generation
- avoid pretending that participating malicious servers can be made harmless
- remove or isolate placeholder admin features until real authorization and audit behavior exist

## Onboarding Strategy Summary

- guided operator setup
- operator chooses naming mode during install
- operator receives explicit federation preflight checklist
- users get Element-first onboarding with homeserver URL and trust caveats
- avoid exposing Matrix jargon unless needed

## UX Strategy Summary

Answer to required question 6:

Element is sufficient for the initial product unless the project later requires custom onboarding UX, tightly integrated server discovery UX, or domestic-network-specific trust guidance that cannot be done through config and docs alone.

Initial UX decision:

- use Element Web and Element mobile clients
- customize branding and onboarding docs only
- defer custom client work until a concrete user-flow blocker is proven

## Operational Safety Summary

- backups and restore must be documented before federation rollout
- upgrades must be staged and reversible
- abuse, moderation, and incident response must be documented for small operators
- operators need a clear warning that federation increases attack surface and metadata exposure
- management scripts must not imply safe destructive actions without backup verification

## Milestone Overview

- Milestone 0: Documentation and reality alignment
- Milestone 1: Single-server hardening baseline
- Milestone 2: Internal naming and trust architecture for full federation
- Milestone 3: Multi-server federation implementation
- Milestone 4: Bootstrap/discovery and operator onboarding system
- Milestone 5: Operational safety, abuse handling, maintainability, and publication readiness

## Current Phase

Phase 2 implementation: deployment-profile and certificate-flow integration

## Next Action

Prepare the repo for the final full-federation architecture using the selected naming and trust profile:

- continue removing or clearly quarantining the remaining stale Dendrite paths and instructions
- implement and document the primary profile: internal DNS plus domains, with public CA preferred when workable
- keep private/internal CA as the real fallback
- keep static host mapping plus private/internal CA as the only supported fallback when DNS fails
- define the exact changes needed to turn on federation safely for a publishable multi-server internal deployment

## Known Risks

- Biggest risk: false confidence from assuming Matrix federation is the main missing feature when the larger problem is naming, trust distribution, operator UX, and operational safety.
- Conduit may or may not remain the best upstream homeserver choice long term; that needs an explicit follow-up assessment.
- Current admin panel is not a trustworthy control plane.
- Current IP-mode HTTPS is not a credible federation story for independent operators.
- Current docs contain materially incorrect statements inherited from Dendrite-era content.
- English README and installer cleanup has started, but README-FA and migration-era content still need review.

## Required Direct Answers

### What is the lightest-weight path to support internal-network federation between independently operated servers?

Keep Zanjir as a packaging base, keep Element, keep a Matrix homeserver, add a documented internal naming and trust model, enable federation intentionally, and test with two operator-managed servers. Solve this mostly with configuration, deployment tooling, and docs, not with a custom protocol or custom client.

Selected primary profile:

- primary: internal DNS plus domains, with public CA preferred when workable
- first fallback: internal DNS plus private/internal CA
- second fallback: static host mapping plus private/internal CA
- not supported for full federation: IP-only self-signed mode
- not supported for full federation: plain HTTP homeserver federation

### What is the safest practical bootstrap model without a strong central point of failure?

Use invite-based joins and known-entry-point introducers. Allow operators to publish a small trusted introducer list, but do not require one global registry for ongoing network function.

### What can and cannot be hidden from a malicious participating server?

Can partly reduce exposure:

- avoid third-party identity servers
- limit public directory exposure
- minimize logs
- reduce unnecessary profile publishing

Cannot realistically hide in Matrix from a malicious participating server:

- that a room or event path involves the malicious server
- the homeserver relationship of users it sees
- communication timing and some membership patterns
- room existence and event flow in rooms it participates in
- large parts of federation metadata needed to route and validate events

### What is the fastest realistic path to a publishable final product?

Keep upstream Matrix clients, keep the packaging concept, clean the docs, define a production-grade internal naming and trust model, harden defaults, implement federation as a first-class deployment mode, and validate the system across multiple independently operated internal servers before publication.

## Final Product Federation Profile

Chosen profile for implementation planning:

- each homeserver gets a stable internal hostname
- operators use valid TLS certificates for those hostnames
- public CA is preferred if domain validation is actually workable
- if public CA is not workable, operators trust a shared internal CA or another explicitly distributed trust root
- Caddy serves certificates for the selected trust model
- Element points to the stable internal homeserver URL
- `.well-known` returns the internal hostname, not a direct IP identity
- federation is enabled only in the federation profile

Supported fallbacks:

- internal DNS with private/internal CA certificates
- static host mapping for each peer hostname plus the same private/internal CA trust model

Explicitly not part of the final federation architecture:

- direct-IP federation as the normal operating model
- public identity server dependency
- mandatory permanent central registry
- plain HTTP between homeservers as a federation transport model

## Exact Full-Federation Change Set

Configuration changes:

- add a documented federation-enabled profile in `docker-compose.yml`
- change `CONDUIT_ALLOW_FEDERATION` from hardcoded false to a profile-controlled value
- remove the baked-in `matrix.org` trust assumption for internal-only deployments
- keep registration and encryption behavior explicit rather than relying on hidden defaults
- document the certificate mode used by the deployment profile

Reverse-proxy and naming changes:

- split Caddy guidance into clear modes:
  - isolated single-server IP mode
  - internal-name federation mode
- update `.well-known` behavior so the server identity matches the internal hostname model
- document when custom ports are acceptable and how peer homeservers will discover them
- document that HTTP may exist only on trusted local backend hops such as reverse-proxy to homeserver, not as inter-homeserver federation

Installer changes:

- add an install-time choice for deployment mode:
  - isolated single-server
  - federated internal-network
- if federated mode is chosen, prompt for:
  - stable internal hostname
  - certificate mode:
    - public CA
    - private/internal CA
  - optional static peer hostname mapping
- stop presenting IP-mode convenience as a federation-grade path
- remove or quarantine dead Dendrite helper paths and stale references

Client configuration changes:

- keep Element as the default client
- leave identity server blank for internal deployments
- document the exact homeserver URL users should enter on web and mobile
- add onboarding text for trusting the selected certificate model where needed

Documentation changes:

- continue rewriting README sections that still point to Dendrite or PostgreSQL
- add a dedicated two-server federation lab guide
- add a federation preflight checklist
- add backup, restore, and rollback steps for the federation profile

Validation steps for the architecture:

1. Install server A with internal hostname and the chosen certificate model.
2. Install server B with the same trust model.
3. Verify name resolution and TLS trust from A to B and B to A.
4. Enable federation profile on both servers.
5. Test invite-based DM across servers.
6. Test encrypted room across servers using Element clients.
7. Test room-alias-based join if alias publication is configured.
8. Add at least one more independently operated server and validate incremental federation learning.
9. Record failure behavior when one server becomes unreachable.

Rollback for staged rollout:

- disable federation profile
- remove peer-facing documentation and discovery entries
- retain server data and trust roots
- restore last known-good compose and proxy templates

### What is the most dangerous source of false confidence?

Believing that enabling a homeserver federation flag solves the product problem. It does not solve naming, TLS trust, operator onboarding, discovery, moderation, abuse handling, recovery, or Matrix metadata limitations.

### What should be solved by configuration vs custom code vs deployment tooling vs documentation?

Configuration:

- federation enablement
- registration defaults
- TURN and reverse-proxy settings
- Element client defaults

Custom code:

- only where the installer or admin UX must enforce safe workflows that config cannot express

Deployment tooling:

- repeatable install profiles
- trust-root distribution
- peer onboarding helpers
- backup/restore automation

Documentation:

- operator decisions
- bootstrap/discovery flows
- internal naming options
- hardening, abuse, moderation, recovery, and upgrade procedures

## Glossary

- Zanjir: this repository; currently a Matrix deployment distribution.
- Matrix: the upstream open messaging protocol and ecosystem.
- Homeserver: the server that stores local user accounts and federates with peers.
- Federation: server-to-server exchange between independent homeservers.
- Bootstrap: how a new server first joins and becomes reachable to other servers.
- Discovery: how servers learn about rooms, aliases, and peers over time.
- Introducer server: a trusted server used to help first contact or publish limited onboarding information.
- Internal naming: hostnames or address conventions usable without public DNS.
- Domestic/internal network: a network environment with limited or no international internet access.
- E2EE: end-to-end encryption between clients; does not hide all metadata from homeservers.

## Next Session Prompt

Read `PROJECT_MASTER.md`, `ARCHITECTURE.md`, `DECISIONS.md`, and `CHAT_HANDOFF.md` first. Then inspect the repo files referenced in those docs, especially `docker-compose.yml`, `README.md`, `install.sh`, `admin/app.py`, `Caddyfile`, and `config/element-config.json`. The selected full-federation profile is `internal DNS + domains`, with `public CA` preferred when workable, `private/internal CA` as the real fallback, and `static host mapping + private/internal CA` as the fallback when DNS fails. Your task is to implement the repo changes for a publishable internal-federation product: remove stale Dendrite-era instructions and dead paths, introduce a federation-capable install/config profile, document multi-server deployment and operator workflows, and keep IP-only mode and plain HTTP explicitly limited to isolated or backend-only use. Update all planning docs after every meaningful conclusion or code change.
