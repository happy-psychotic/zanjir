# TASKS

Status markers:

- `[todo]`
- `[doing]`
- `[done]`
- `[blocked]`

## Priority Queue

| Priority | Status | Task | Owner | Dependencies | Notes |
|---|---|---|---|---|---|
| P0 | [done] | Reconcile README and migration docs with actual Conduit runtime | TBD | Phase 0 docs | README, README-FA, MIGRATE, and MIGRATE-FA now reflect the Conduit-based runtime and current install modes |
| P0 | [done] | Decide and document supported domestic-network naming/trust profiles | TBD | Phase 0 docs | Selected: internal DNS + domains, public CA preferred, private/internal CA fallback |
| P0 | [doing] | Define exact federation-enable change set for the publishable full product | TBD | naming/trust profile | Installer, proxy templates, and Element config now support deployment profiles; validation and runbooks still remain |
| P0 | [done] | Review and constrain admin panel trust model | Codex | strict `ADMIN_USERS` allowlist | Replaced placeholder admin authz with explicit Matrix ID allowlist for `/admin` |
| P1 | [todo] | Document backup, restore, and upgrade runbooks | TBD | Phase 1 | Must exist before federation rollout |
| P1 | [done] | Remove or quarantine dead Dendrite installer paths and unused config | TBD | README reconciliation | Dead Dendrite installer functions and helper files removed in first implementation pass |
| P1 | [done] | Decide how to remove `matrix.org` assumptions from isolated/internal deployments | TBD | naming/trust profile | Default compose trusted-server assumption changed from `matrix.org` to empty list |
| P1 | [todo] | Create operator onboarding flow for first federated peer join | TBD | federation lab plan | Prefer invite and known-entry-point workflow |
| P2 | [todo] | Evaluate whether current homeserver choice remains the best long-term upstream base | TBD | architecture plan stabilized | Do not block current cleanup on this unless a hard blocker appears |
| P2 | [todo] | Define abuse, moderation, and incident response guidance | TBD | federation lab validated | Focus on small trusted operators first |

## Short-Term Tasks

- `[done]` Inspect repository and document current reality.
- `[done]` Create root durable planning and handoff docs.
- `[done]` Turn required analysis topics into a concrete implementation-ready plan.
- `[done]` Produce a line-item list of repo files that must change for full federation.
- `[todo]` Update the remaining wording and tasks that still say MVP where the final-product intent should be explicit.
- `[done]` Implement the documentation cleanup for Dendrite-era drift.
- `[done]` Define the concrete env/config switches for isolated mode vs federation mode.
- `[done]` Identify the main config assumptions that break in an internal CA deployment and replace them with explicit PEM-file handling in Caddy and the installer.
- `[todo]` Verify whether public CA issuance is actually possible for the intended internal domains.
- `[todo]` Define the private/internal CA distribution workflow if public CA is not workable.
- `[done]` Remove migration-only markdown guides that no longer add operational value.

## Medium-Term Tasks

- `[done]` Add installer support for choosing naming/trust profile.
- `[todo]` Add documented preflight checks for federation readiness.
- `[todo]` Build a two-server test harness or repeatable lab instructions.
- `[done]` Implement explicit admin authorization checks with an `ADMIN_USERS` allowlist.
- `[todo]` Document room alias, invite, and peer onboarding procedures for operators.

## Task Notes By Theme

### Federation

- Required answer: Zanjir already inherits Matrix federation capability from the homeserver stack, but the active config disables it.
- Missing work is mostly operational and configuration work, not a new protocol subsystem.

### Naming

- Public DNS must not be assumed.
- Selected full-federation profile is stable internal names with public CA preferred when workable.
- Private/internal CA is the real fallback.
- Static hosts-file distribution is acceptable only when DNS fails.
- IP-only self-signed mode is not a supported full-federation profile.
- Internal domains are acceptable and preferred if available on the internal network.
- Plain HTTP is not a supported homeserver federation fallback.

### Bootstrap and Discovery

- Prefer invite-based and introducer-based bootstrapping.
- Do not plan around a permanent central registry.

### Client

- Element is sufficient unless a proven operator or user blocker appears.

### Security and Privacy

- Do not promise metadata privacy against participating malicious servers.
- Audit and logging strategy must avoid leaking secrets or sensitive event content unnecessarily.

### Implementation Progress

- English README cleanup has started.
- Dead Dendrite installer logic and helper files have been removed.
- Default `matrix.org` trusted-server assumption has been removed from compose defaults.
- Compose now supports env-driven federation enablement via `FEDERATION_ENABLED`.
- Installer now writes `FEDERATION_ENABLED` and `CERT_MODE` to `.env`.
- Installer now writes `DEPLOYMENT_MODE`, `PUBLIC_BASE_URL`, and `WELL_KNOWN_SERVER` to `.env`.
- Caddy now has separate public-CA, private/internal-CA, and isolated IP-mode deployment templates.

## Risks / Blockers Linked To Tasks

- `[blocked]` Long-term homeserver-base recommendation remains incomplete until upstream suitability is reassessed separately.
- `[todo]` Federation enablement still needs a concrete certificate distribution workflow and install UX.
- `[blocked]` Admin panel work should not expand until its trust boundary is explicitly defined.
