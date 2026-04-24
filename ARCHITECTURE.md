# ARCHITECTURE

## Current-State Architecture

Zanjir currently packages a single-node Matrix stack:

- `conduit` as the homeserver in [docker-compose.yml](/home/saeid/Documents/Codex/Zanjir/docker-compose.yml:6)
- `element` as the web client in [docker-compose.yml](/home/saeid/Documents/Codex/Zanjir/docker-compose.yml:72)
- `caddy` as TLS reverse proxy and static web serving in [docker-compose.yml](/home/saeid/Documents/Codex/Zanjir/docker-compose.yml:83)
- `coturn` for VoIP NAT traversal in [docker-compose.yml](/home/saeid/Documents/Codex/Zanjir/docker-compose.yml:29)
- a custom Flask `admin` service in [docker-compose.yml](/home/saeid/Documents/Codex/Zanjir/docker-compose.yml:55)

Current behavior is isolated-server oriented because federation is disabled in the active homeserver env config.

## Component Map

Upstream components:

- Conduit homeserver
- Element Web
- Caddy
- coturn

Custom repository components:

- installer script
- CLI management wrapper
- Caddy templates for domain and IP modes
- Element branding/config overrides
- small admin web app

Legacy or stale components:

- `dendrite/` config and key-generation paths
- migration docs
- README sections still describing Dendrite commands and PostgreSQL

## What Is Upstream vs Custom

Reusable as-is:

- Element as the initial client
- basic container packaging approach
- Caddy reverse-proxy pattern
- coturn inclusion

Needs modification:

- federation profile
- naming/trust setup
- installer prompts and outputs
- docs and operational guidance
- admin panel authorization model

Should likely be replaced or reduced:

- dead Dendrite-era code paths
- misleading admin functionality that is only placeholder logic

## Current-State Issues

- `CONDUIT_ALLOW_FEDERATION` is set to false.
- `.well-known` responses were previously tied to the old public-domain path; they now follow installer-generated `PUBLIC_BASE_URL` and `WELL_KNOWN_SERVER` values.
- admin panel authorization is not real; `check_if_admin()` currently returns true for any successful login in [admin/app.py](/home/saeid/Documents/Codex/Zanjir/admin/app.py:121).
- audit logging is local SQLite only and not tied to authoritative server-side admin actions.
- the installer previously collapsed certificate handling into a simple domain-versus-IP split; it now supports explicit isolated/federated deployment mode and public/private certificate mode selection.

## Does Zanjir Already Get Matrix Federation "For Free"?

Yes at the protocol capability level, no at the product level.

Protocol level:

- the chosen homeserver already implements Matrix federation behavior when enabled
- no custom federation protocol needs to be invented

Product level:

- federation is disabled today
- domestic-network naming and trust are not designed
- onboarding and discovery are not designed
- operational safety for federation is not documented

## Target-State Architecture

Target architecture remains a Matrix-based distribution:

- one homeserver stack per operator
- Element clients reused
- reverse proxy terminates trusted certificates appropriate to the internal environment
- federation enabled for known peers
- bootstrap and discovery rely on invitations, room aliases, known peers, and optional introducers
- docs define trust boundaries and operational procedures

This is lighter and safer than building custom messaging infrastructure.

## Implementation Status Update

Implemented in the repo:

- `docker-compose.yml` now mounts `./certs` into Caddy and passes `PUBLIC_BASE_URL`, `WELL_KNOWN_SERVER`, and TLS file env vars.
- `install.sh` now selects between `isolated` and `federated` mode, blocks federation-grade assumptions from IP-only installs, writes deployment metadata into `.env`, and copies private/internal CA PEM files into `./certs`.
- `Caddyfile` now represents the public-CA domain path, `Caddyfile.private-ca` represents the operator-provided PEM path, and `Caddyfile.ip-mode` is explicitly limited to isolated/test use.
- `.well-known` responses and Element config now follow generated deployment values instead of hardcoded public-domain assumptions.

## Federation Design

Decision:

Use upstream Matrix federation behavior; do not build custom federation code unless a specific gap is proven later.

Practical federation model:

- servers federate only when operators intentionally expose them to each other
- homeservers learn peers incrementally through room joins, invites, and aliases
- public room usability is supported, but not via a mandatory global registry
- cross-server encrypted private rooms and DMs rely on standard Matrix E2EE behavior in clients

Key missing work:

- safe enablement profile
- naming/trust distribution
- peer connectivity guide
- abuse and moderation posture

## Bootstrap / Join Design

Preferred bootstrap path:

1. Operator installs a local homeserver using a supported naming/trust profile.
2. Operator receives a small list of trusted introducer servers or direct peer contacts.
3. New server joins via one of:
   - invite from an existing room member
   - known room alias published by a trusted peer
   - direct peer/operator introduction
4. Server learns additional peers incrementally through normal Matrix participation.

Why this model:

- avoids dependence on one permanent global registry
- keeps first contact understandable
- aligns with Matrix room and invite mechanics

## Discovery Design

Discovery is not treated as universal decentralized search.

Chosen discovery primitives:

- room participation
- invitations
- known peer list
- optional trusted introducer list
- optional room-directory publication on trusted servers

What is intentionally not promised:

- automatic global peer discovery
- hidden federation topology
- robust anonymous discovery

## Internal Naming Design

Recommended profiles:

### Profile A - Internal DNS + Public CA

Preferred when workable.

- each homeserver gets a stable internal hostname
- certificates are issued by a publicly trusted CA if domain validation works in the actual environment
- `.well-known` and client config can use stable names
- easiest operator experience when it can be made to work honestly

### Profile B - Internal DNS + Private/Internal CA

Primary fallback when public CA is not workable.

- each homeserver still gets a stable internal hostname
- operators distribute and trust a private/internal CA
- maintains the same naming model with a different trust chain

### Profile C - Static Host Mapping + Private/Internal CA

Fallback when DNS is weak and the only supported fallback once DNS is unavailable.

- operators distribute host mapping entries and trust roots
- workable for small networks
- higher operator burden and more error-prone

### Profile D - Direct Address / IP Fallback

Last resort only.

- acceptable for testing or very small controlled deployments
- poor certificate UX and weak long-term maintainability
- not recommended as the core federation architecture
- excluded from the normal federation path

## Selected Federation Profile

Chosen implementation target:

- internal DNS plus domains, with public CA preferred when workable

Supported fallback:

- internal DNS plus private/internal CA
- static host mapping plus private/internal CA

Explicitly excluded from the full federation design:

- direct-IP self-signed federation as a normal operating mode
- plain HTTP federation between homeservers

Rationale:

- gives each server a stable identity
- uses public trust when possible without assuming it will always work
- preserves a viable private trust model if public PKI is not available
- is easier to support consistently than ad hoc IP trust
- keeps operator workflows explicit and auditable

## Deployment Model for Domestic/Internal Network

Assumptions:

- no international internet
- partial routing isolation
- inconsistent DNS conditions
- mixed operator skill

Recommended deployment model:

- retain containerized install path
- support preloaded or mirrored images
- support internal registry mirror configuration
- avoid third-party identity server dependency
- publish explicit required ports and routing expectations
- define operator runbooks for trust-root and peer-address distribution

Reference multi-server deployment shape:

- server A: `matrix-a.example.internal`
- server B: `matrix-b.example.internal`
- server C: `matrix-c.example.internal`
- servers resolve each other's internal names through internal DNS
- servers use either public CA certificates or a shared private/internal CA
- servers expose Matrix client and federation endpoints through Caddy over HTTPS
- Element users authenticate only against their own homeserver

Domains are acceptable and preferred in this project if they are available on the internal network, because they produce a more stable federation identity and a more supportable certificate model than raw IP addressing.

## Encryption and Identity Considerations

What Matrix with Element gives:

- mature client-side E2EE for private rooms and DMs where clients support it
- reuse of existing Element apps and web client

What it does not solve:

- full metadata privacy from homeservers
- hiding the homeserver relationship of users from peers that interact with them
- hiding room participation from servers that must process room events

Identity guidance:

- avoid external identity servers for the product by default
- keep 3PID integrations disabled unless a real need appears
- prefer local accounts only

## Malicious-Server Considerations

A malicious participating homeserver can often observe:

- membership and join/leave timing in rooms it participates in
- event flow timing
- sender homeservers
- some room topology and alias information
- media and federation access patterns reaching it

It cannot be assumed harmless just because message content may be E2EE between clients.

Mitigations:

- federate first with trusted operators
- minimize public exposure
- document trust and moderation boundaries
- keep logs lean
- do not expose unnecessary admin APIs

## Metadata Exposure Analysis

Important realism:

- Matrix is not a strong metadata-hiding system.
- Participating servers learn meaningful routing and relationship metadata.
- Homeservers can often infer who talks to whom, when, and through which rooms.
- Room existence cannot be hidden from servers that participate in those rooms.
- Federation participants necessarily see other homeserver names involved in event exchange.

This project should market privacy as:

- self-hosting and reduced dependence on external providers
- local operator control
- E2EE for supported room/message flows

It should not market:

- anonymity
- strong unlinkability
- hidden social graph against malicious federation participants

## Moderation / Abuse Implications

Federation introduces:

- remote abuse sources
- spam invitations
- moderation coordination challenges
- trust and block-list decisions across operators

Initial operating posture:

- federate only with trusted peers first
- document room and server blocking procedures
- define operator escalation and incident response basics before broad rollout

## Operator Model

Primary operator types:

- family or household operator
- small community operator
- trusted organization or cooperative operator

Design bias:

- simple guided install
- explicit supported profiles
- clear warnings when a weaker profile is selected
- minimal requirement to understand Matrix internals

Domains are acceptable and preferred in this project if they are available on the internal network, because they produce a more stable federation identity and a more supportable certificate model than raw IP addressing.

## Client Strategy

Initial client strategy:

- Element Web remains the web client
- Element mobile apps remain the recommended native apps
- keep identity server blank or disabled
- use branding and onboarding docs before any custom client work

Custom client work is avoidable unless federation onboarding UX proves too confusing inside Element alone.

## Exact Repo Change Areas For Full Federation

`docker-compose.yml`:

- replace hardcoded federation-off behavior with profile-driven configuration
- remove internal-only reliance on `matrix.org` as a trusted server assumption

`install.sh`:

- add deployment mode selection
- add prompts for internal hostname and trust profile
- stop implying IP mode is sufficient for federation
- remove or quarantine dead Dendrite functions and references

`Caddyfile` and `Caddyfile.ip-mode`:

- keep IP mode for isolated/test use only
- add or document a federation-capable internal-name mode
- ensure `.well-known` values match the chosen hostname model
- document that reverse-proxy to homeserver may use local HTTP, but federation must remain HTTPS/TLS

`config/element-config.json`:

- keep homeserver base URL aligned to internal hostname
- keep identity-server-related behavior disabled for internal deployments
- add documented guidance for CA trust on web and mobile clients

`README.md` and related docs:

- remove Dendrite-era account creation and PostgreSQL claims
- add federation preflight, multi-server deployment guide, and rollback guidance

`admin/app.py`:

- do not expand trust in this component until real authorization checks exist
- document that it is not the federation control plane

## What Parts Need Modification

- compose/config templates for federation-enabled mode
- reverse-proxy and naming guidance
- installer UX for trust and naming profile selection
- docs and safety runbooks
- admin panel authz model or scope

## What Parts Should Be Replaced

- stale Dendrite commands and docs
- stale PostgreSQL references
- any admin action surfaced as real if it is placeholder-only

## Open Technical Questions

- Is Conduit the right long-term upstream homeserver base for this project, or should the project evaluate a different Matrix homeserver after the current architecture and implementation work?
- What is the most supportable trust model for certificates in the target domestic environment: internal CA, private PKI, or pinned peer trust?
- Which federation-related settings in the chosen homeserver need explicit hardening beyond a simple enable flag?
- How much operator automation should be added before it starts hiding critical trust decisions?
- Should the admin panel be expanded, minimized, or removed from the trusted path?
