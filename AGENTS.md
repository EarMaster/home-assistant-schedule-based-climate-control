# AGENTS.md

Detailed guidance for AI agents working in this repository.

## Project Overview

This is a single-file Home Assistant blueprint (`schedule_based_climate_control.yaml`) for automating climate devices (ACs, thermostats, heaters) based on schedules and sensor states. There is no build system, package manager, or test infrastructure — the entire deliverable is one YAML file.

## Repository Structure

- `schedule_based_climate_control.yaml` — the blueprint; the only file users import into Home Assistant
- `README.md` — user-facing docs; also the authoritative source for the current version number
- `.github/workflows/release.yaml` — CI that auto-bumps the patch version and creates a GitHub release

## Versioning

The version string lives in **two places** and must stay in sync:

| File | Pattern |
|------|---------|
| `README.md` | `**Current Version:** X.Y.Z` |
| `schedule_based_climate_control.yaml` | `*Version:* X.Y.Z` |

The CI workflow reads the version from `README.md`, increments the patch digit, and updates both files automatically on every push to `main` that touches `schedule_based_climate_control.yaml`. When editing the blueprint manually, **do not pre-bump the version** — the CI handles it.

## Release Flow

Pushing a change to `schedule_based_climate_control.yaml` on `main` triggers the workflow:
1. Extracts current version from `README.md`
2. Increments patch (`1.1.5` → `1.1.6`)
3. Updates version strings in both `README.md` and `schedule_based_climate_control.yaml`
4. Commits with message `Bump version to vX.Y.Z [skip ci]`
5. Creates a GitHub release tagged `vX.Y.Z`

## Blueprint Logic

The automation uses `mode: restart` and triggers on state changes of all configured entities (temp sensor, schedule, vacation/party mode booleans, window sensors).

**Decision hierarchy in `action`:**

1. **Safety cut-off** (turn OFF): vacation mode ON, any window open (after delay), or schedule inactive
2. **Party mode** (stop/no-op): automation exits without touching devices
3. **Automatic control**: compares `current_temp` against `limit_hot` / `limit_cold` thresholds and calls `climate.set_temperature` with the appropriate `hvac_mode` (`cool` or `heat`), or turns off if temp is already in the comfort band

**Hybrid inputs**: Each temperature value has an optional entity selector (e.g. `target_temp_cool_ent`) and a fallback static number (`target_temp_cool`). The Jinja variable resolution pattern is:
```yaml
{% if tc_ent != none and states(tc_ent) | is_number %} {{ states(tc_ent) | float }}
{% else %} {{ tc_num | float }} {% endif %}
```

## Editing the Blueprint

- The blueprint is pure YAML — validate with a YAML linter before committing.
- Minimum HA version is set in `blueprint.homeassistant.min_version`; only raise it when using features introduced in a newer release.
- New optional inputs should include a `default:` to preserve backwards compatibility with existing automations.
- The `source_url` field in the blueprint header must point to the raw GitHub URL on `main`; do not change it.
