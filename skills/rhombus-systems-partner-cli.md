---
name: rhombus-partner-cli
description: >
  Rhombus CLI reference for MSP and reseller partners managing multiple client
  organizations. Covers partner authentication, the `--partner-org` flag,
  `rhombus partner get-partner-clients-v2`, the active-client pattern
  (.claude/rhombus-partner.local.md), and multi-org / cross-client workflows.
  Use whenever the user operates across clients — multi-tenant audits,
  cross-org alerts, fleet-wide device inventories, per-client reporting, or
  MSP/reseller day-to-day work. Does NOT duplicate CLI basics (install, auth,
  auto-generated commands) — for those, see the user plugin's `rhombus-cli`
  skill.
argument-hint: "[command or topic]"
allowed-tools: Read, Grep, Glob, Bash
---

# Rhombus CLI — Partner Edition

Partner-specific usage of the `rhombus` CLI. For core CLI setup (install, auth, global flags, auto-generated API commands), see the `rhombus-cli` skill in the user plugin — this skill only covers what is different for partners.

## Partner authentication

The `rhombus login` browser flow supports partner accounts. When you authenticate with a partner identity, you gain access to every client organization that identity can manage.

```bash
rhombus login   # use your partner credentials in the browser
```

Verify you're authenticated as a partner:

```bash
rhombus partner get-partner-clients-v2
```

If this returns a non-empty list, you're a partner. If you get a 403 or empty list, log in again with partner credentials.

## The `--partner-org` flag

`--partner-org` is the core of partner workflows. Append it to **any** command to target a specific client org.

```bash
# By name (fuzzy matched, case-insensitive)
rhombus camera get-minimal-camera-state-list --partner-org "acme corp"

# By UUID
rhombus camera get-minimal-camera-state-list --partner-org "abc123..."

# Recent alerts for a client
rhombus alert recent --partner-org "acme corp" --max 10

# Analyze footage on a client's camera
rhombus analyze footage "front lobby" --partner-org "acme corp" --period "last hour"
```

Without `--partner-org`, commands run against **your own partner org** (usually a management-only org with no cameras). Most partner work requires the flag.

## Active-client state (this plugin)

Typing `--partner-org "acme corp"` on every command is repetitive. This plugin supports an **active-client** concept stored in `.claude/rhombus-partner.local.md`:

```markdown
---
active_client: acme corp
---

# Partner workspace notes

(Any markdown notes you want to keep per workspace.)
```

- Written by the `/rhombus-client-switch` command.
- Read by the `rhombus-partner-session-init` hook (announces on SessionStart) and the `rhombus-client-selector` agent (uses as default when a client reference is ambiguous).
- The `rhombus-cli-validate` hook nudges you if you run a client-scoped command without `--partner-org` and without an active client.

Precedence: if you pass `--partner-org` explicitly, it always wins. The active-client is only used when you don't.

## Listing client orgs

```bash
rhombus partner get-partner-clients-v2
```

Returns JSON with `partnerClients[].orgName` and `partnerClients[].orgUuid`. Use this to pick a client, feed into fleet-wide loops, or build a client picker UI.

The `/rhombus-clients` slash command wraps this with table formatting.

## Multi-org recipes

### Batch op across every client

```bash
rhombus partner get-partner-clients-v2 | jq -r '.partnerClients[].orgName' | while IFS= read -r org; do
  echo "=== $org ==="
  rhombus camera get-minimal-camera-state-list --partner-org "$org"
done
```

### Find offline cameras across the fleet

```bash
rhombus partner get-partner-clients-v2 | jq -r '.partnerClients[].orgName' | while IFS= read -r org; do
  offline=$(rhombus camera get-minimal-camera-state-list --partner-org "$org" \
    | jq '[.cameraStates[] | select(.connectionState != "connected")] | length')
  if [ "$offline" -gt 0 ]; then
    echo "$org: $offline offline cameras"
  fi
done
```

### Cross-org alert volume

```bash
rhombus partner get-partner-clients-v2 | jq -r '.partnerClients[].orgName' | while IFS= read -r org; do
  count=$(rhombus alert recent --partner-org "$org" --max 100 2>/dev/null | jq '.alerts | length')
  [ "$count" -gt 0 ] && echo "$org: $count alerts in last hour"
done
```

These recipes are wrapped in the `/rhombus-audit-clients` slash command and the `rhombus-fleet-ops` agent.

## Slash commands and agents in this plugin

| Component | Purpose |
|---|---|
| `/rhombus-clients` | List managed orgs |
| `/rhombus-client-switch` | Set active client (writes `.claude/rhombus-partner.local.md`) |
| `/rhombus-audit-clients` | Multi-org audit (offline cameras, alert volume, storage, license) |
| `/rhombus-client-alerts` | Alerts for one client |
| `/rhombus-client-devices` | Device inventory for one client |
| `/rhombus-fleet-report` | Weekly/monthly cross-client report |
| `rhombus-client-selector` agent | Resolves ambiguous client references |
| `rhombus-fleet-ops` agent | Runs multi-org audits and produces reports |
| `rhombus-client-onboarding` agent | Walks new-client onboarding |

## Troubleshooting

| Issue | Fix |
|---|---|
| `get-partner-clients-v2` returns empty | You're not logged in as a partner — re-run `rhombus login` with partner credentials |
| 403 with `--partner-org` | Verify the client name matches exactly (case-insensitive, substring); use the UUID to disambiguate |
| Commands hit your own org by accident | Set an active client with `/rhombus-client-switch <name>` |
| The active-client isn't being picked up | Check `.claude/rhombus-partner.local.md` exists at the repo root and has `active_client:` in its frontmatter |

## When to also enable `rhombus-developer`

If you're writing custom integrations for a client (e.g., a bespoke webhook receiver, a client-specific dashboard), enable the `rhombus-developer` plugin alongside `rhombus-partner`. The developer plugin provides MCP tools, SDK codegen, and webhook scaffolding. The partner plugin does not overlap — it focuses on CLI + multi-org workflows.
