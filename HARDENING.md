<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--send-google-chat-webhook/v0.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--send-google-chat-webhook/v0.0.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct `${{ }}` expression interpolation inside `run:` shell commands in the 'download binary' step. `${{ env.VERSION }}` and `${{ env.BINARY_NAME }}` are interpolated directly into the curl URL and tar command strings before the shell processes them. Even though these env vars are set to literal values in the same step's `env:` block, any `${{ ... }}` inside a `run:` block is a script-injection risk because YAML template substitution happens before shell quoting. Offending lines: `curl -LOv "https://...v${{ env.VERSION }}/..."` and `tar xzf ${{ env.BINARY_NAME }}_${{ env.VERSION }}_...`. Additionally, rule (b): the `tar xzf` line is unquoted, allowing shell metacharacter injection from the expanded value.

Locations:

- `action.yml:53`
- `action.yml:54`

### script-injection (severity: high)

Rule (a): Direct `${{ inputs.webhook_url }}` expression interpolation inside a `run:` shell command in the 'send message via cli' step. The `inputs.webhook_url` value is attacker-controlled and is interpolated directly into the shell command string: `./send-google-chat-webhook chat workflownotification --webhook-url="${{ inputs.webhook_url }}"`. This allows an attacker supplying a crafted webhook URL to inject arbitrary shell commands.

Locations:

- `action.yml:58`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.webhook_url }}" appears directly in run: block of step "send message via cli"; move to env: map

Locations:

- `action.yml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all three script-injection findings in hardened/action/action.yml:
1. 'download binary' step (lines 53-54): Replaced `${{ env.VERSION }}` and `${{ env.BINARY_NAME }}` with plain shell variable references `${VERSION}` and `${BINARY_NAME}`. Also added double-quotes around the tar argument to prevent shell metacharacter injection.
2. 'send message via cli' step (lines 58/62): Moved `${{ inputs.webhook_url }}` out of the run: block into the env: block as `WEBHOOK_URL: ${{ inputs.webhook_url }}`, and updated the run: block to reference it as `"$WEBHOOK_URL"` instead of the inline expression.

