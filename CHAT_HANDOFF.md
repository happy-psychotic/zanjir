# CHAT_HANDOFF

## What This Project Is

Zanjir is currently a Matrix deployment distribution built around Conduit, Element Web, Caddy, coturn, and a small custom Flask admin panel. It is not a new protocol or a full federation platform yet.

## What Was Done In This Session

- cloned and inspected the real repository
- reviewed runtime stack, installer, proxy config, Element config, admin app, and top-level docs
- documented that active runtime uses Conduit while multiple docs still assume Dendrite
- documented that Matrix federation is inherited from the homeserver but currently disabled by config
- created durable planning, roadmap, architecture, decisions, and task docs
- removed dead Dendrite installer functions and helper files
- updated the English README to stop pointing at Dendrite-only account creation and PostgreSQL-era backup guidance
- removed the default `matrix.org` trusted-server assumption from `docker-compose.yml`
- changed federation enablement from hardcoded-off to env-controlled in `docker-compose.yml`
- installer now writes `FEDERATION_ENABLED` and `CERT_MODE` into `.env`
- installer now prompts for `isolated` versus `federated` mode, plus `public` versus `private` certificate mode for domain installs
- private/internal CA mode now copies operator-provided PEM files into `./certs` and activates `Caddyfile.private-ca`
- Caddy `.well-known` responses and Element config now follow installer-generated `PUBLIC_BASE_URL` and `WELL_KNOWN_SERVER` values
- README-FA and migration guides were cleaned up to remove stale Dendrite/PostgreSQL operational instructions

## Read First Next Session

1. `PROJECT_MASTER.md`
2. `ARCHITECTURE.md`
3. `DECISIONS.md`
4. `ROADMAP.md`
5. `TASKS.md`

Then inspect:

- `docker-compose.yml`
- `README.md`
- `install.sh`
- `admin/app.py`
- `Caddyfile`
- `config/element-config.json`

## Current Phase

Phase 2 implementation: deployment-profile and certificate-flow integration

## Next Exact Task

Implement the planned full-federation change set in the repo:

- continue reconciling the remaining stale Dendrite-era docs
- introduce the selected federation profile: internal DNS plus domains, with public CA preferred when workable
- keep private/internal CA as the real fallback
- keep static host mapping plus private/internal CA as the fallback when DNS fails
- specify and then apply the exact config and documentation changes required for a publishable multi-server internal federation deployment

Progress on that task:

- implemented the real installer/config flow for isolated versus federated installs
- implemented certificate-mode mapping in Caddy for public CA versus operator-provided private/internal CA
- kept IP-only mode explicitly limited to isolated/test use
- cleaned the main remaining migration-era and Persian documentation drift

## Blockers / Open Questions

- Long-term homeserver choice is not yet reassessed beyond repo inspection.
- Public CA viability for the intended internal domains is still not verified.
- Private/internal CA distribution workflow is only implemented as local PEM-file installation; shared trust-root distribution and peer onboarding docs still need a fuller runbook.
- Admin panel trust model is weak and should not be treated as authoritative yet.

## Strict Reminder

Update `PROJECT_MASTER.md`, `ROADMAP.md`, `ARCHITECTURE.md`, `DECISIONS.md`, `TASKS.md`, and this file after every meaningful conclusion or implementation change.

## Next Session Prompt

Read `PROJECT_MASTER.md`, `ARCHITECTURE.md`, `DECISIONS.md`, and `CHAT_HANDOFF.md` first. Then inspect `docker-compose.yml`, `README.md`, `install.sh`, `admin/app.py`, `Caddyfile`, and `config/element-config.json`. The selected full-federation profile is `internal DNS + domains`, with `public CA` preferred when workable, `private/internal CA` as the real fallback, and `static host mapping + private/internal CA` as the fallback when DNS fails. Dendrite installer dead paths have already been removed, the English README has begun cleanup, the default `matrix.org` trust assumption has already been removed, and federation enablement is now env-controlled via `FEDERATION_ENABLED`. Your task is to continue implementation: finish the remaining doc cleanup, complete the federation-capable install/config profile, document multi-server deployment and operator workflows, and keep IP-only mode and plain HTTP explicitly limited to isolated or backend-only use. Update all planning docs after every meaningful conclusion or code change.
