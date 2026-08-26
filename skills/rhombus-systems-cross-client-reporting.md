---
name: rhombus-cross-client-reporting
description: >
  Produce fleet-wide health and activity reports across every managed Rhombus
  client organization — weekly health summaries, monthly executive reports,
  incident rollups, license utilization, alert volume trends, camera uptime.
  Use whenever the user asks for a "fleet report", "MSP report", "client
  summary", "weekly health across clients", "monthly rollup", a report
  spanning multiple orgs, or wants a shareable artifact about the state of
  their managed fleet. Produces markdown reports that can be emailed, pasted
  into a ticket, or committed to a knowledge base.
argument-hint: "[weekly|monthly|incident] [optional: client filter]"
allowed-tools: Read, Bash, Write
---

# Cross-Client Reporting

Build shareable reports spanning every managed client org. Reports are generated in markdown; if the user wants PDF or HTML, recommend `pandoc` after generation.

## Report templates

See `references/report-templates.md` for three templates:

1. **Weekly health** — offline cameras, alert volume, door controller issues, battery-low sensors, per-client.
2. **Monthly executive** — trend lines (alert volume week-over-week), license utilization, top 5 noisy clients, top 5 healthy clients.
3. **Incident rollup** — one client, one window. Used after a notable event for an internal write-up.

## Data sources

All data comes from the CLI (from the `rhombus-partner-cli` skill):

- Client list: `rhombus partner get-partner-clients-v2`
- Cameras: `rhombus camera get-minimal-camera-state-list --partner-org <client>`
- Doors: `rhombus door get-minimal-door-state-list --partner-org <client>`
- Alerts: `rhombus alert recent --partner-org <client> --max 500 --after "7d ago"`
- Licenses: `rhombus license get-license-state --partner-org <client>`

For speed, loop over `partner-clients-v2` and aggregate in memory before rendering.

## Process

1. **Confirm scope** — which report template? Which clients (default: all)? Which time window (default: last 7 days for weekly, last 30 for monthly)?
2. **Collect data** — loop every client, running the queries above in parallel where possible (`xargs -P`).
3. **Render** — fill the appropriate template from `references/report-templates.md`. Include:
   - A headline summary (one sentence, one or two key numbers)
   - Per-client table
   - Callouts for outliers
   - Recommended actions (ranked by impact)
4. **Output** — print to stdout by default; if the user specifies an output path, `Write` to that file.

## Delegation

For the actual collection work, you can delegate to the `rhombus-fleet-ops` agent — it has an optimized loop and knows how to recover from per-client failures (e.g., a single client with auth issues shouldn't kill the whole report).

## Output conventions

- **File naming:** `fleet-report-<weekly|monthly>-<YYYY-MM-DD>.md`
- **Formatting:** headline numbers in the opening summary; detail tables below; trend arrows (↑/↓/→) for week-over-week or month-over-month comparisons.
- **Confidential data:** client names, camera names, and alert details. Warn the user before sharing outside their org.

## Edge cases

- **A client has 0 cameras.** Skip from health tables; note in a footer.
- **A client's API access is broken.** Show "—" with a footnote; don't crash the report.
- **You have >200 clients.** Chunk the loop; report can take minutes. Offer a progress indicator.
- **The user wants only specific clients.** Respect their filter list rather than defaulting to "all".

## Follow-up suggestions

After rendering:

- "Commit this to your shared Knowledge Base"
- "Email it to your customer success team"
- "Drill into [top-alerting-client] with `/rhombus-client-alerts <name> 7d`"
