---
name: rhombus-cli
description: >
  Rhombus CLI reference — installation, authentication, profiles, global flags,
  and the hand-written `rhombus login`, `rhombus configure`, `rhombus alert`,
  and `rhombus footage` commands, plus the ~60 auto-generated API service groups
  (camera, door, user, access-control, etc.). Use when the user asks how to
  install or configure the CLI, wants to run a rhombus command, asks about
  profiles or credentials, needs help finding an auto-generated CLI command
  (like `rhombus camera get-minimal-camera-state-list`), wants to pull recent
  alerts, or open live footage. Does NOT cover: `rhombus context/analyze/stitch`
  (see rhombus-deployment-context skill) or `rhombus chat/voice` (see
  rhombus-mind skill).
argument-hint: "[command or topic]"
allowed-tools: Read, Grep, Glob, Bash
---

# Rhombus CLI

The `rhombus` CLI wraps the entire Rhombus REST API into a single binary. Source: https://github.com/RhombusSystems/rhombus-cli.

This skill covers installation, auth, global flags, and the hand-written commands that don't deserve their own skill. Related skills:

- **`rhombus-deployment-context`** — `rhombus context`, `analyze`, `stitch`
- **`rhombus-mind`** — `rhombus chat`, `voice`
- **`rhombus-support-links`** — support.rhombussystems.com article directory

## Installation

```bash
# Homebrew (macOS)
brew install RhombusSystems/tap/rhombus

# Shell script (macOS/Linux)
curl -fsSL https://raw.githubusercontent.com/RhombusSystems/rhombus-cli/main/install.sh | sh

# PowerShell (Windows)
irm https://raw.githubusercontent.com/RhombusSystems/rhombus-cli/main/install.ps1 | iex
```

## Authentication

Two methods — browser-based (recommended) or manual:

```bash
# Browser-based OAuth2 login (creates permanent API key automatically)
rhombus login

# Manual configuration
rhombus configure
# Prompts for: API Key, Output format, Endpoint URL
```

### Profiles

Multiple named profiles are supported (like AWS CLI):

```bash
rhombus configure --profile staging
rhombus alert recent --profile staging
```

### Environment variables

| Variable | Purpose |
|---|---|
| `RHOMBUS_API_KEY` | API key |
| `RHOMBUS_PROFILE` | Active profile name |
| `RHOMBUS_OUTPUT` | Output format (json/table/text) |
| `RHOMBUS_ENDPOINT_URL` | API base URL |

### Config files

Stored in `~/.rhombus/` as INI format:

- `~/.rhombus/config` — settings (output format, endpoint)
- `~/.rhombus/credentials` — API keys (file mode 0600)
- `~/.rhombus/certs/<profile>/` — client certs for mTLS auth

## Global flags

Every command accepts these:

| Flag | Description |
|---|---|
| `--profile` | Config profile (default: "default") |
| `--output` | Output format: json, table, text |
| `--api-key` | Override API key for this call |
| `--endpoint-url` | Override API base URL |
| `--version` | Show CLI version |

## Hand-written commands (covered here)

### `rhombus login`
Browser-based OAuth2 PKCE authentication. Opens browser, receives callback on `localhost:11434`, creates a permanent API key. Supports partner accounts. 5-minute timeout.

### `rhombus configure`
Interactive setup: API key, output format, endpoint URL. Use `--profile` for multi-profile configs.

### `rhombus alert`

| Subcommand | Description | Key flags |
|---|---|---|
| `alert recent` | List policy alerts from last hour | `--camera` (name/UUID), `--after` ("1h ago", "5m ago", epoch ms), `--max` (default 20) |
| `alert thumb [uuid]` | Download alert thumbnail JPEG | `--output` (file path) |
| `alert download [uuid]` | Download alert video clip | `--output` (file path) |
| `alert play [uuid]` | Play alert clip in browser | — |

Camera names are fuzzy-matched (case-insensitive substring).

### `rhombus footage [camera]`
Opens a Rhombus camera player in the browser. Defaults to live view. Use `--start` to jump to a specific time in the past.

```bash
rhombus footage "front lobby"
rhombus footage "front lobby" --start "5m ago"
rhombus footage "front lobby" --token-duration 7200   # 2-hour session
```

## Auto-generated API commands

~60 service groups, each with multiple subcommands mapping to Rhombus API endpoints. Every generated command follows the same pattern:

```bash
rhombus <service-group> <operation> [flags]

# Examples
rhombus camera get-minimal-camera-state-list
rhombus door get-minimal-door-state-list
rhombus user get-users-in-org
rhombus event get-policy-alerts-v2 --after-timestamp-ms 1700000000000
```

### Key patterns

**Discover available operations:**
```bash
rhombus camera --help
rhombus access-control --help
```

**Generate a JSON skeleton with all parameters:**
```bash
rhombus camera get-camera-config --generate-cli-skeleton
```

**Pass parameters as JSON:**
```bash
# Inline JSON
rhombus camera get-camera-config --cli-input-json '{"cameraUuid":"cam_123"}'

# From file
rhombus camera get-camera-config --cli-input-json file://params.json
```

**Flags override JSON input.** **All flags are kebab-case** (e.g., `--camera-uuid`). **All API calls are POST.**

### Service groups

See `references/commands.md` for the full 60+ list. Major categories:

- **Devices:** camera, sensor, door, door-controller, doorbell-camera, audiogateway, audioplayback, elevator, climate, badge-reader, button, ble, media-device, device-config
- **Access Control:** access-control, access-control-integrations, guest-management-kiosk
- **AI & Analytics:** face-recognition-person, face-recognition-event, face-recognition-matchmaker, vehicle, occupancy, logistics, proximity, scene-query, search
- **Events & Monitoring:** event, event-search, alert-monitoring, alarm-monitoring-keypad, lockdown-plan, policy, rules, rules-records, schedule, rapidsos
- **Organization:** org, user, user-metadata, customer, location, permission, license, feature, partner
- **Integrations:** integrations, webhook-integrations, org-integrations, iot-integrations, storage-integrations, incident-management-integrations, service-management-integrations, oauth, developer
- **Media & Reports:** video, upload, export, report, tvos-config, help-service

See `references/workflows.md` for common recipes.

## Troubleshooting

| Issue | Fix |
|---|---|
| `command not found: rhombus` | Run install again or check PATH includes `/usr/local/bin` |
| Auth errors | Run `rhombus login` or check `~/.rhombus/credentials` |
| Wrong org data | Check `--profile` flag or `RHOMBUS_PROFILE` env var |
| Login timeout | Ensure browser can reach `console.rhombussystems.com` and callback port 11434 is free |

## API base URL

Default: `https://api2.rhombussystems.com`. Override per-command with `--endpoint-url` or globally via `rhombus configure`.

## Relationship to the `rhombus-developer` plugin

This skill is for day-to-day CLI use. If you need direct API integration (HTTP calls, SDK generation, OpenAPI spec, webhooks, MCP tools), enable the `rhombus-developer` plugin. The CLI wraps the same API — every auto-generated CLI command maps 1:1 to an API endpoint.
