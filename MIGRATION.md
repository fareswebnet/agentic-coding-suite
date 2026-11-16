# Migration notes

## 0.5.0

Project renamed from `sdlc-rebar` to `agentic-coding-suite`; the plugin is now called `coding`, and the `sdlc-` prefix is gone from skill names.

- Plugin: `/plugin install coding@agentic-coding-suite`; skills run as `/coding:<name>` (was `/rebar:sdlc-<name>`).
- Standalone install (`scripts/install.py`): folders are `coding-<name>` (was `sdlc-<name>`), invoked as `/coding-<name>` or `$coding-<name>`. Delete the old `sdlc-*` folders from your skills directory.
- `skills/sdlc-<name>/` is now `skills/<name>/`, and `evals/sdlc-<name>.json` is now `evals/<name>.json`.
- Native role subagents are written as `.claude/agents/coding-<role>.md` (was `sdlc-<role>.md`); delete the old files.
- Old GitHub URLs redirect to `fareswebnet/agentic-coding-suite`.

## 0.4.0

Skill renames (old names on the left). Reinstall and delete the old folders from your skills directory (`~/.claude/skills`, `~/.codex/skills`, `.agents/skills`, ...), otherwise both versions stay installed.

| Old | New |
|---|---|
| `sdlc-delivery-system` | `delivery-orchestrator` |
| `sdlc-code-review-quality-gate` | `code-review` |
| `sdlc-code-review-reception` | `receiving-code-review` |
| `sdlc-data-database-architecture` | `data-architecture` |
| `sdlc-quality-test-strategy` | `test-strategy` |
| `sdlc-devops-ci-cd-platform` | `ci-cd-platform` |
| `sdlc-documentation-knowledge` | `documentation` |
| `sdlc-ai-assisted-development` | `ai-coding-governance` |
| `sdlc-performance-scalability-cost` | `scalability-cost` |
| `sdlc-system-merge-convergence` | `system-merge` |
| `sdlc-product-analytics-experimentation` | `product-analytics` |

Removed (no measured gain, see `docs/references/skill-value-analysis.md`): `sdlc-product-strategy` (use `idea-framing`), `sdlc-api-integration-contracts` (use `api-versioning-deprecation`), `sdlc-identity-access-multitenancy` (use `security-privacy-compliance`), `sdlc-release-production-readiness` (use `definition-of-done`), `sdlc-technical-debt-evolution` (use `delivery-retrospective`), `sdlc-maturity-assessment`, `sdlc-skill-authoring` (use `skill-evaluation`).

Other changes:

- `--native-roles` now writes `.claude/agents/sdlc-<role>.md`; delete the old unprefixed files (renamed again in 0.5.0, see above).
- `presets/preset-<name>.md` is now `presets/<name>.md`.
