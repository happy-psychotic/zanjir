# DECISIONS

## ADR-0001 - Use Zanjir As the Initial Base

Status:

Accepted provisionally

Context:

The repository already packages a working Matrix stack with installer, reverse proxy, client, TURN, and branding. Replacing it immediately would skip the required assessment step and lose packaging value already present.

Decision:

Use Zanjir as the initial base, but treat it as a distribution shell around upstream Matrix components, not as an architecture that must be preserved unchanged.

Alternatives considered:

- discard Zanjir immediately and start a new distribution
- treat Zanjir as a complete product foundation

Consequences:

- short-term work focuses on cleanup, docs, and safe federation enablement
- long-term homeserver choice can still be revisited

## ADR-0002 - Federation Is Inherited, Not a New Feature To Invent

Status:

Accepted

Context:

The active runtime is a Matrix homeserver stack, and the compose config explicitly disables federation rather than lacking a federation-capable protocol path.

Decision:

Treat federation as an upstream capability that must be enabled and operationalized, not as a custom protocol feature that Zanjir must newly implement.

Alternatives considered:

- build custom federation logic
- describe federation as absent from current architecture

Consequences:

- most work shifts to naming, TLS trust, onboarding, moderation, and ops
- documentation must stop implying that federation requires a new subsystem

## ADR-0003 - Bootstrap Model Uses Minimal Centralization

Status:

Accepted

Context:

The target environment should tolerate failure or blocking of any one server and should not depend on a permanent global registry.

Decision:

Bootstrap through invite-based joins, known-entry-point operators, room aliases where workable, and optional trusted introducer servers.

Alternatives considered:

- single mandatory central registry
- fully automatic global discovery
- manual-only direct peering without documented introducers

Consequences:

- first contact is simple enough for operators
- no single introducer becomes structurally required
- discovery remains practical rather than magical

## ADR-0004 - Discovery Model Is Incremental Federation Learning

Status:

Accepted

Context:

In real Matrix-style deployments, peers usually learn about one another through room participation and invitations rather than omniscient discovery.

Decision:

Base discovery on room participation, invitations, known peers, and an optional trusted introducer list.

Alternatives considered:

- mandatory global room registry
- custom decentralized discovery protocol

Consequences:

- lower implementation burden
- clear operator mental model
- discovery remains partial and context-driven by design

## ADR-0005 - Prefer Stable Internal Names Over IP-Only Federation

Status:

Accepted

Context:

Public DNS and public CA infrastructure may be missing, but independently operated federation still needs stable peer identity and transport trust.

Decision:

Prefer internal DNS with domains for the final product. Prefer public CA certificates when workable, use private/internal CA as fallback, and treat direct-IP mode as testing or last resort rather than the primary federation design.

Alternatives considered:

- IP-only deployment as the default federation mode
- requiring public DNS and public CA

Consequences:

- stronger and more supportable peer identity model
- more setup work than simple IP mode
- clearer long-term maintainability

## ADR-0009 - Full Federation Uses Internal DNS With Public CA Preferred

Status:

Accepted

Context:

The project needs a specific final-product implementation target for domestic-network federation. The repo already has an IP-only mode, but that mode is operationally weak for independent operators and does not provide a durable trust model. The deployment environment now confirms internal DNS and domains are available.

Decision:

Use `internal DNS + domains` as the federation base. Prefer `public CA` certificates when domain validation is actually workable. Use `private/internal CA` as the real fallback. Support `static host mapping + private/internal CA` only when DNS fails. Keep IP-only mode limited to isolated or test scenarios.

Alternatives considered:

- IP-only self-signed mode as the normal federation path
- requiring public DNS and public CA
- plain HTTP federation between homeservers
- delaying any profile selection until implementation time

Consequences:

- installer and docs must ask operators for stable internal names
- certificate mode selection becomes a first-class deployment concern
- federation documentation becomes much clearer and less misleading

## ADR-0011 - Do Not Use Plain HTTP For Homeserver Federation

Status:

Accepted

Context:

The target environment may not always support public CA issuance, but Matrix federation still relies on HTTPS/TLS semantics rather than plain HTTP for inter-homeserver operation.

Decision:

Do not treat plain HTTP as a fallback for homeserver federation. HTTP may be used only on trusted local backend hops such as reverse-proxy to homeserver communication inside one deployment.

Alternatives considered:

- allow plain HTTP between homeservers when certificate issuance fails
- build a separate non-standard federation mode

Consequences:

- certificate management is mandatory for real federation
- deployment docs must explain certificate fallback honestly
- backend HTTP remains acceptable only inside one operator's local stack

## ADR-0010 - Prefer Final-Product Planning Over MVP Framing

Status:

Accepted

Context:

The project direction has been corrected: the goal is not to stop at an MVP. The goal is to design and build toward a publishable, complete product suitable for presentation and release.

Decision:

Plan toward the final publishable product now, while still executing in safe phases. Do not frame core architectural decisions as temporary MVP-only choices unless they are explicitly provisional.

Alternatives considered:

- retain MVP-first framing
- defer final-product architecture decisions until after a lab prototype

Consequences:

- docs, roadmap, and task sequencing must describe the full intended system
- phased execution remains useful, but phases are now steps toward the final product rather than a reduced-scope end state

## ADR-0006 - Use Element As The Initial Client Strategy

Status:

Accepted

Context:

The repo already integrates Element Web, and the product goal prefers low client-development cost.

Decision:

Use Element Web and Element mobile apps for the product. Defer custom client work unless there is a proven blocker that config, branding, or onboarding docs cannot solve.

Alternatives considered:

- build a custom client immediately
- fork Element immediately

Consequences:

- faster path to a publishable product
- inherits Matrix client strengths and weaknesses
- domestic-network onboarding may need supplemental docs

## ADR-0007 - Documentation Is a First-Class Control Surface

Status:

Accepted

Context:

This project has substantial operator-facing complexity. Unsafe implicit knowledge would create brittle future sessions and brittle deployments.

Decision:

Maintain root-level durable docs as the authoritative planning and handoff layer. Every meaningful architectural conclusion or implementation change must update the docs.

Alternatives considered:

- keep planning in chat only
- scatter planning across ad hoc notes

Consequences:

- future sessions can resume without lost context
- doc drift becomes easier to detect
- implementation work must carry documentation overhead intentionally

## ADR-0008 - Solve The Product Primarily With Config, Tooling, And Docs

Status:

Accepted

Context:

The biggest gaps identified are naming, trust, onboarding, hardening, and operational guidance, not message transport logic.

Decision:

Prefer configuration changes, deployment tooling, and operator documentation before writing new application code.

Alternatives considered:

- build a large custom control plane first
- build a custom client first

Consequences:

- lower maintenance burden
- faster path to a publishable product
- forces realism about what Matrix already does and does not provide
