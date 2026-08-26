# Changelog

All notable changes to nodus-a2a-wire are documented here.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

---

## [0.1.0] — 2026-08-26

First publish. The package existed and was complete since 2026-05-29 but **could
not be published**: it declared the distribution name `nodus-a2a`, which is taken
by a different first-party package.

### Changed

- **Distribution renamed `nodus-a2a` -> `nodus-a2a-wire`**, matching the repo.

- **Python module renamed `nodus_a2a` -> `nodus_a2a_wire`.** Renaming the
  distribution alone was not enough: the published `nodus-a2a` coordinator ships
  a module *also* called `nodus_a2a`, so installing both wrote one over the
  other. Measured before the fix — installing this package on top of the
  published coordinator left `AgentCoordinator`, `AgentRegistry` and
  `DeadLetterService` gone, with pip reporting success. Both now coexist.

- **`nodus-lang` removed from `dependencies`.** Nothing here imports it: a host
  constructs `A2AHttpServer` and wires it to their own
  `NodusRuntime.tool_registry`, so the runtime arrives by injection. Verified by
  installing with `--no-deps` into a clean venv and importing the full API with
  nodus-lang absent. It is now a `dev` extra, for the one test that needs it.

  The declaration had also pinned `nodus-lang>=4.0.0,<5.0.0`, which would have
  excluded every 5.x release and violated the no-caps policy. The suite passes
  188/188 against nodus-lang 5.5.0.

- **Project URLs** now point at `Masterplanner25/nodus-a2a-wire` rather than the
  coordinator's repo.

See [nodus-lang#477](https://github.com/Masterplanner25/Nodus/issues/477).

---

## [0.1.0-unpublished] — 2026-05-29

Initial release — prepare-not-publish (coordinated three-artifact launch with
nodus-lang 4.0.0 and nodus-mcp 0.1.0).

### Added

- **A2A 1.0.0 message-only server** over HTTP+JSON/REST (`A2AHttpServer`).
- **std:tool → AgentSkill projection** (`project_skill`, `build_agent_card`):
  tool name, description, tags, and metadata examples projected to AgentSkill;
  deprecated tools excluded.
- **Agent Card serving** at `/.well-known/agent-card.json` (1.0 well-known URI;
  `agent.json` 0.3-era path returns 404).
- **Part type dispatch**: tool results automatically dispatched to the correct
  A2A Part variant — `str` → TextPart, `bytes` → RawPart (base64), all other
  JSON-serializable values → DataPart.
- **Tool-call-envelope dispatch** via `DataPart(data={"tool": "<name>",
  "args": {...}})` with single-tool fallback for single-tool agents.
- **Bearer auth** (`HTTPAuthSecurityScheme`) — token validator callable in
  `ServerConfig`; dev mode allows all requests when validator is not set.
- **A2A-Version negotiation** — lenient on missing header, strict on mismatch
  (`VersionNotSupportedError` → HTTP 400); matches Major.Minor, accepts patch.
- **Error packaging** — tool exceptions are returned as an error DataPart in an
  HTTP 200 response (application errors never become HTTP 5xx).
- **snake_case ↔ camelCase codec** throughout (`mediaType`, `messageId`,
  `contextId`, `protocolBinding`, etc.); proto field names never leak to wire.
- **8 standing assertions** (`test_invariants.py`) covering no-new-opcodes,
  no-task-emitted, no-kind-discriminator, no-legacy-wellknown, version-
  negotiation, codec-name-mapping, capability-honesty, and inversion-note.
- **169 tests** (unit, transport, integration) with 93% source coverage.
- **CLI entry-point** (`python -m nodus_a2a_wire serve`) for smoke-testing
  connectivity and Agent Card serving.

### Design decisions

- **D5 (message-only archetype)**: server never emits a Task in v0.1. All
  task-management operations return `UnsupportedOperationError` (HTTP 501).
- **D6 (inversion note)**: A2A `INPUT_REQUIRED` is the park-and-resume model.
  The nodus-mcp no-thread-parks rule must NOT be imported into a2a v0.2 Task
  lifecycle. This is documented in `docs/design/05-deferred-features.md §2`.
- **BYTECODE_VERSION**: stays 4. Zero new nodus-lang opcodes introduced.

### Deferred to v0.2+

Task lifecycle, streaming (`SendStreamingMessage`), push notifications, Agent
Card signing (D8a), extended/authenticated card (D8b), JSON-RPC binding,
gRPC, OAuth2/OIDC/mTLS, tenant routing, 0.3 wire-dialect compatibility.
See `docs/design/05-deferred-features.md` for the full inventory.

[0.1.0]: https://github.com/Masterplanner25/nodus-a2a-wire/releases/tag/v0.1.0
