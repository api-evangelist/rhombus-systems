---
name: rhombus-partner-onboarding
description: >
  Onboarding runbook for adding a new client organization to partner
  management. Covers provisioning API access, inviting the partner admin,
  setting retention policy, configuring alert routing, and validating the
  onboarding with a smoke test. Use whenever the user says "add a new
  client", "onboard an org", "new MSP client", "set up a new managed org",
  "client X is going live", or asks to walk through the steps for bringing
  a client under partner management.
argument-hint: "[client-name]"
allowed-tools: Read, Bash, Write
---

# Partner Client Onboarding

Runbook for bringing a new client organization under partner management. Every step is a CLI call; the checklist lives in `references/onboarding-checklist.md`.

## Prerequisites

- You are authenticated as a partner (`rhombus partner get-partner-clients-v2` returns your client list).
- The client organization already exists in Rhombus (i.e., the client has spun up their own Rhombus org via the Rhombus Console, or Rhombus sales has provisioned it).
- You have the client admin's email ready (for the partner-admin invite).

## High-level flow

1. **Confirm the client org is visible** — `rhombus partner get-partner-clients-v2 | grep -i "<client-name>"`. If not present, ask the user to reach out to Rhombus to attach the client to their partner account.
2. **Set as active client** — `/rhombus-client-switch <client-name>`. Every subsequent command scopes to this client by default.
3. **Provision the partner API key** for the client — this is what the partner uses to automate against the client. The `rhombus-cli` auto-generates one on `rhombus login`; for scripted usage, use the Developer Webservice.
4. **Invite the partner admin** to the client org with the role that matches your engagement (usually "Admin" for full MSP, "Viewer" for monitoring-only).
5. **Set retention and alert policies** — bring them in line with your MSP's standard baseline. Ship the baseline as a saved JSON at `references/onboarding-checklist.md` so every client starts identical.
6. **Configure alert routing** — webhooks to your monitoring platform (PagerDuty, Slack, custom), email lists, and escalation rules.
7. **Validate with smoke tests** — pull recent alerts, fetch camera list, simulate a test alert, confirm it lands in your monitoring.
8. **Document the onboarding** in your client-ops log.

## Walkthrough

Parse `$ARGUMENTS` as the client name. If missing, ask.

Then for each step in `references/onboarding-checklist.md`:

1. Announce the step.
2. Run the command.
3. Show the output, flag any red flags.
4. Ask the user if they want to continue, make a change, or abort.

End with a summary report of every action taken, ready to paste into the client-ops log.

## When to prefer the slash command

`/rhombus-clients` to discover the client, `/rhombus-client-switch` to activate, then invoke this skill by asking Claude to "onboard <client name>". The `rhombus-client-onboarding` agent wraps this into a guided flow.

## Edge cases

- **Client already has an existing partner integration.** Don't clobber. Ask the user how to handle it (override, side-by-side, abort).
- **Retention policy conflicts with a client's existing configuration.** Always surface the diff before applying — never silently overwrite retention settings.
- **Webhook URL not yet provisioned.** Note as a gap; complete the rest of onboarding so you can deliver a working-baseline-minus-one-thing.
