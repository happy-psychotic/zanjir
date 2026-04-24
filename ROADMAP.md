# ROADMAP

## Phase 0 - Reality Alignment

Objective:

Establish one truthful set of docs and decisions for what Zanjir is today.

Why it exists:

The repo currently mixes Conduit runtime with stale Dendrite instructions and isolated-server assumptions. Planning on top of that would be unsafe.

Prerequisites:

- repository inspection complete

Concrete deliverables:

- root planning docs created
- stale assumptions listed
- current architecture and gaps documented

Likely code areas or files affected:

- `README.md`
- `README-FA.md`
- `install.sh`
- `docker-compose.yml`

Test strategy:

- documentation review against actual repo files
- line-by-line claim validation for core runtime components

Rollback strategy:

- git revert doc-only changes

Exit criteria:

- no core planning doc contradicts the active compose stack

## Phase 1 - Single-Server Hardening Baseline

Objective:

Make the current single-server mode safer and more internally consistent before federation work.

Why it exists:

Federating an operationally weak baseline multiplies risk.

Prerequisites:

- Phase 0 complete

Concrete deliverables:

- remove or quarantine dead Dendrite config paths
- review exposed ports and defaults
- document backup and restore
- document safe upgrade path
- review admin panel trust and authorization gaps

Likely code areas or files affected:

- `docker-compose.yml`
- `install.sh`
- `zanjir-cli.sh`
- `admin/app.py`
- `README.md`

Test strategy:

- fresh local install
- backup and restore dry run
- admin login and non-admin access tests
- log review for secret leakage

Rollback strategy:

- restore previous compose and installer templates
- keep volume backups before migration

Exit criteria:

- documented backup/restore
- documented upgrade path
- known admin-panel risks either fixed or clearly warned

## Phase 2 - Naming and Trust Architecture

Objective:

Define supported domestic-network naming and trust models for federation.

Why it exists:

Federation depends on stable peer identity and transport trust more than on application code.

Prerequisites:

- Phase 1 complete

Concrete deliverables:

- supported deployment profiles:
  - internal DNS + public CA
  - internal DNS + private/internal CA
  - isolated IP mode for test use only
- operator docs for each profile
- configuration matrix showing when each profile is acceptable

Likely code areas or files affected:

- `install.sh`
- `Caddyfile`
- `Caddyfile.ip-mode`
- new docs and example config files

Test strategy:

- two-node lab validation per profile
- certificate and peer connectivity verification
- failure injection for DNS or host mapping mistakes

Rollback strategy:

- revert to single-server isolated profile
- retain old trust material until cutover succeeds

Exit criteria:

- one recommended profile selected for the final product
- one fallback profile documented with explicit limitations
- IP-only federation explicitly rejected as the normal final-product architecture

Current status:

- profile selection is now implemented in `install.sh`
- proxy templates now map certificate mode to an actual deployment path
- remaining work is validation, runbooks, and peer-onboarding docs

## Phase 3 - Full Federation Implementation

Objective:

Enable and validate usable internal-network federation between independently operated servers.

Why it exists:

This turns Zanjir from an isolated deployment kit into a publishable federated base distribution.

Prerequisites:

- Phase 2 complete

Concrete deliverables:

- federation-enabled compose/config profile
- multi-server deployment guide
- room join, invite, alias, and DM scenarios documented
- explicit list of unsupported assumptions and limitations
- rollback steps back to isolated mode documented

Likely code areas or files affected:

- `docker-compose.yml`
- `install.sh`
- `config/element-config.json`
- reverse-proxy config
- operator docs

Test strategy:

- server A invite to server B
- server B invite to server C
- public room alias join across servers
- direct message across servers
- encrypted room test with Element clients
- offline/peer-loss behavior observation
- partial-network and resolver-failure behavior observation

Rollback strategy:

- toggle back to isolated mode
- preserve server data and trust roots
- remove peer exposure docs and config if rollout fails

Exit criteria:

- multiple independently operated servers can federate on an internal network using the chosen naming/trust profile with documented operator procedures

## Phase 4 - Bootstrap and Discovery Workflows

Objective:

Design the least-centralized practical server onboarding model.

Why it exists:

Federation without documented peer bootstrap and discovery produces fragile operator experiences.

Prerequisites:

- Phase 3 complete

Concrete deliverables:

- invite-based server introduction workflow
- known-entry-point onboarding workflow
- optional trusted introducer list format
- room-alias-based discovery guidance

Likely code areas or files affected:

- docs
- installer prompts or templates if needed
- optional helper scripts

Test strategy:

- new operator joins from scratch using docs only
- onboarding success test without global registry

Rollback strategy:

- fall back to manual peer configuration workflow

Exit criteria:

- a non-expert operator can join an existing internal federation using documented steps

## Phase 5 - Operational Safety and Abuse Controls

Objective:

Add the procedures needed to run a small federated network safely.

Why it exists:

Federation increases abuse, moderation, and incident-response burden.

Prerequisites:

- Phase 4 complete

Concrete deliverables:

- moderation model for small operators
- spam and abuse handling guidance
- incident response checklist
- recovery and key-loss guidance
- logging and privacy guidance

Likely code areas or files affected:

- docs
- admin panel or helper scripts if justified

Test strategy:

- tabletop exercises for abuse and outage cases
- restore-from-backup drill

Rollback strategy:

- temporarily limit federation scope to trusted peers only

Exit criteria:

- operators can respond to common failure and abuse scenarios with documented procedures

## Phase 6 - Publication Readiness

Objective:

Refine UX, docs, packaging, and deployment ergonomics without breaking the architecture.

Why it exists:

The project should become easier to operate only after the base model is sound.

Prerequisites:

- Phase 5 complete

Concrete deliverables:

- cleaner installer flow
- clearer status and diagnostics
- optional onboarding helpers
- trimmed custom admin surface
- publication-ready documentation set
- installer and deployment defaults suitable for external users of the project

Likely code areas or files affected:

- `install.sh`
- `zanjir-cli.sh`
- `admin/`
- docs

Test strategy:

- fresh operator install trial
- documentation-only install test

Rollback strategy:

- preserve previous stable installer path

Exit criteria:

- the final product can be presented, published, deployed, and operated by a small non-expert team with docs and standard Matrix clients
