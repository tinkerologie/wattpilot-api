# AI Agent Instructions for Wattpilot API Python Library

> Single source of truth for all AI coding assistants working on this repository.

## Project Identity

| Key | Value |
| --- | --- |
| **Name** | Wattpilot API Python Library |
| **Repository** | `github.com/tinkerologie/wattpilot-api` |
| **Default Branch** | `main` |
| **Description** | Python asynchronous library for communicating with Fronius Wattpilot EV chargers. |

The library talks to the Fronius Wattpilot Wallbox via a reverse-engineered WebSocket API. It
provides an async WebSocket client, an interactive CLI shell, an MQTT bridge, and Home Assistant
MQTT discovery. Fully typed, 100% test coverage.

## Mandatory Development Workflow for AI Agents

All AI coding assistants (Claude Code, Gemini, GitHub Copilot, Cursor, Roo, Devin, etc.) MUST strictly adhere to the following workflow when working on this repository:

1. Issue-First Requirement: Never start writing code or creating pull requests without an existing GitHub Issue.
2. Issue Template: Issues must be created using the official Issue Forms (`01-bug-report.yml` for bugs or `02-enhancement-request.yml` for enhancements).
3. CodeRabbit AI Plan: Every issue must have a generated Coding Plan from CodeRabbit AI (`@coderabbitai plan`). AI agents must read and implement strictly according to the approved plan.
4. Branch Discipline: Always create and work on a dedicated branch (`feat/<issue-id>-<description>` or `fix/<issue-id>-<description>`). NEVER commit directly to `main`.
5. Documentation & Changelog: Update `CHANGELOG.md` under `## [Unreleased]` for any user-facing or architectural changes.
6. Local Verification: Run all test, lint, format, and typecheck commands locally before committing. All tests must pass with zero errors and zero warnings.
7. PR Template Completion: Always use `.github/PULL_REQUEST_TEMPLATE.md`. Complete all fields, link the issue using `Closes #<id>` or `Fixes #<id>` when the PR fully resolves it, or `Part of #<id>` for partial work (only maintainers with write access may write `None`, for housekeeping), and check all Pre-Submission Governance boxes with real terminal output.
8. Draft PR First: ALWAYS open pull requests as DRAFT (`gh pr create --draft`).
9. CI Verification: Wait for CI workflows to run on the draft PR and verify that all status checks pass.
10. Ready for Review: Only mark the PR as ready for review (`gh pr ready`) once all checks pass and all due diligence is verified.
11. Review Clearance: Review and resolve all CodeRabbit AI and GitHub Copilot comments. Ensure Codecov reports show zero errors, zero warnings, and no coverage drop.


## Executable Verification Commands

- `pytest` — Run test suite
- `pytest --cov=src` — Run tests with coverage
- `ruff check .` — Run ruff linter
- `ruff format --check .` — Check code formatting
- `mypy src` — Static type checking
- `pre-commit run --all-files` — Run all configured pre-commit hooks

```bash
# Install in development mode (with dev dependencies)
pip install -e ".[dev]"

# Run tests with coverage (must reach 100%)
pytest --cov=wattpilot_api --cov-report=term-missing --cov-fail-under=100

# Run integration tests against a real device
WATTPILOT_HOST=<ip> WATTPILOT_PASSWORD=<pass> pytest -m integration -v

# Run the interactive shell
WATTPILOT_HOST=<ip> WATTPILOT_PASSWORD=<pass> wattpilotshell

# Enable MQTT bridge and Home Assistant discovery
MQTT_ENABLED=true MQTT_HOST=<broker> HA_ENABLED=true wattpilotshell
```

## Architecture

### Source Layout

```text
src/wattpilot_api/
├── __init__.py          # Public API exports, __version__
├── client.py            # Async Wattpilot WebSocket client
├── auth.py              # Authentication (PBKDF2 + bcrypt)
├── models.py            # Enums (IntEnum), dataclasses
├── exceptions.py        # Exception hierarchy
├── definition.py        # YAML API definition loader
├── mqtt.py              # Async MQTT bridge (aiomqtt)
├── discovery.py         # Home Assistant MQTT discovery
├── shell.py             # Async CLI shell (prompt_toolkit)
├── py.typed             # PEP 561 marker
└── resources/
    └── wattpilot.yaml   # Property metadata (3300+ lines)
```

### Key Modules

- **`client.py`** — `Wattpilot` class: async context manager, WebSocket connection, PBKDF2/bcrypt auth, property synchronization, callbacks (sync + async)
- **`auth.py`** — Password hashing (PBKDF2/bcrypt), auth response computation, HMAC message signing
- **`models.py`** — `LoadMode`, `CarStatus`, `AccessState`, `ErrorState`, `CableLockMode` enums; `MqttConfig`, `HaConfig`, `DeviceInfo` dataclasses
- **`exceptions.py`** — `WattpilotError` → `ConnectionError`, `AuthenticationError`, `PropertyError`, `CommandError`
- **`definition.py`** — Loads `wattpilot.yaml`, validates structure, splits child properties
- **`mqtt.py`** — `MqttBridge` class + value encoding/decoding helpers
- **`discovery.py`** — `HomeAssistantDiscovery` class for MQTT discovery protocol
- **`shell.py`** — `WattpilotShell` async CLI using prompt_toolkit

### WebSocket Message Flow

`hello → authRequired → auth → authSuccess → fullStatus → deltaStatus`

The `Wattpilot` class connects via `websockets`, authenticates (PBKDF2 default, bcrypt for Wattpilot Flex devices), then receives `fullStatus` followed by `deltaStatus` messages. Property changes propagate via registered callbacks.

### Dependencies

- `websockets>=14.0` — async WebSocket client
- `pyyaml>=6.0` — YAML API definition
- `aiomqtt>=2.0` — async MQTT bridge
- `bcrypt>=4.0` — bcrypt authentication
- `prompt-toolkit>=3.0` — async CLI shell

## Key Conventions

- **Python >=3.12** — uses match/case, modern type hints, `type` aliases
- **Fully async** — all IO uses `async`/`await`, compatible with Home Assistant
- **Property keys** are short codes from the go-eCharger API (e.g., `amp`, `alw`, `nrg`, `lmo`)
- **Configuration via environment variables** — prefixed `WATTPILOT_*`, `MQTT_*`, `HA_*`
- **Value mapping** uses `valueMap` dicts in `wattpilot.yaml` to translate numeric ↔ human-readable
- **MQTT topics** use template placeholders: `{baseTopic}`, `{propName}`, `{serialNumber}`, `{component}`, `{uniqueId}`
- **No global mutable state** — all state is in class instances

## Modifying Charger Properties

1. Update `wattpilot.yaml` — add/modify the property entry (key, jsonType, rw, title, description, valueMap, homeAssistant block, childProps)
2. If adding a named property accessor, add a `@property` in `client.py` and update `_update_property()`
3. Include a `homeAssistant` block with `component` and `config` for HA discovery support
4. Add tests — coverage must remain at 100%

## Boundaries and Safety Constraints

- Always: Write tests for any new or modified functionality, verify changes locally, and update CHANGELOG.md under `[Unreleased]`.
- Ask first: Major architectural changes, removing public APIs, or adding heavy external dependencies.
- Never: Commit secrets, credentials, API keys, or private certificates. Never commit directly to `main`.

## Issue and Pull Request Governance

These rules apply to every contributor, including AI coding agents. The full org-wide rules are in
[GOVERNANCE.md](https://github.com/tinkerologie/.github/blob/main/GOVERNANCE.md).

1. **Issue first.** Open an issue with the bug or enhancement template before opening a pull
   request. Keep the title prefix `[Bug]:` or `[Enhancement]:`, followed by a space and the title.
   Automation applies the `bug` or `enhancement` label when it can tell the type, and CodeRabbit
   then posts a Coding Plan on the issue. Implement from that plan. The labeler skips issues labeled
   `question`, `duplicate`, `invalid`, `wontfix`, or `no-type`, and never re-adds a type label that
   a maintainer removed.
2. **Link the issue.** In the PR's **Related Issues** section, write `Closes #<number>` (or `Fixes`
   or `Resolves`) when the PR completely resolves the issue; these keywords close it on merge. Use
   `Part of #<number>` or `Related to #<number>` for partial work. The linked issue must exist; an
   issue in another public repository (`owner/repo#123`) is fine. Only authors with write access or
   higher (maintainers) may write `None`, for housekeeping such as CI,
   documentation, or dependency updates; PR governance rejects `None` from everyone else.
3. **One change at a time.** Contributors without write access can have at most 2 open, non-draft
   pull requests here. GitHub enforces this limit, and draft pull requests do not count toward it.
   Do not open PRs that edit the same files as another open PR, such as `CHANGELOG.md`,
   translations, `README.md`, or generated bundles. Wait for the first to merge, rebase, then
   continue.
4. **Act on the Tinkerologie-Bot report.** When PR governance fails, Tinkerologie-Bot posts one comment
   listing every problem as a numbered item. Fix each item by editing the PR description or pushing
   the missing change; the check re-runs and the comment updates to passed.
5. **Rebase before review.** Rebase on `main` before marking a PR ready for review.
6. **Resolve conflicts first.** When a PR gets the `conflicts` label, rebase on `main`,
   resolve the conflicts (keep every `CHANGELOG.md` entry under `[Unreleased]`), re-run the tests,
   and force-push before doing any other work.
7. **Coverage.** `codecov/patch` must pass: at least 90% of added or changed lines must be
   covered by tests. The gate is the percentage; still, add tests for the uncovered lines the
   Codecov PR comment lists, and explain in the PR description any line you leave uncovered. Do
   not lower coverage targets or add coverage ignore rules to make it pass.
8. **Stale items.** Issues and PRs with no activity for 14 days are labeled `stale` and closed 30
   days later unless there is new activity. These labels are exempt:
   - Issues: `pinned`, `security`, `confirmed`, `work-in-progress`, `priority: critical`, `priority: high`.
   - Pull requests: `pinned`, `security`, `work-in-progress`.
