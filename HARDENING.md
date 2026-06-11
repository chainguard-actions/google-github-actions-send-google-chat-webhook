<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--send-google-chat-webhook/v0.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **google-github-actions--send-google-chat-webhook/v0.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct expression interpolation inside `run:` shell commands. In the 'download binary' step, `${{ env.VERSION }}` and `${{ env.BINARY_NAME }}` are interpolated directly into shell commands (a `curl` URL and a `tar` invocation). These `env.*` context values flow through YAML template substitution before the shell ever sees them, allowing an attacker who controls the calling workflow's env context to inject arbitrary shell commands. Offending lines:
  `curl -LOv "https://github.com/.../v${{ env.VERSION }}/send-google-chat-webhook_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz"`
  `tar xzf ${{ env.BINARY_NAME }}_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz`
Fix: move VERSION and BINARY_NAME into the `env:` block and reference them as quoted shell variables (e.g. `"$VERSION"`).

Locations:

- `action.yml:44`
- `action.yml:45`

### script-injection (severity: high)

Rule (a): Direct expression interpolation inside a `run:` shell command. In the 'send message via cli' step, `${{ inputs.webhook_url }}` is interpolated directly into the shell command as a CLI argument value:
  `./send-google-chat-webhook chat workflownotification --webhook-url="${{ inputs.webhook_url }}"`
Because `inputs.webhook_url` is supplied by the caller, an attacker can inject arbitrary shell metacharacters (e.g. a webhook URL containing `"; malicious-command; "`) before the shell ever parses the string. Fix: move `inputs.webhook_url` into the `env:` block (e.g. `WEBHOOK_URL: ${{ inputs.webhook_url }}`) and reference it as a quoted shell variable (`"$WEBHOOK_URL"`).

Locations:

- `action.yml:49`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.webhook_url }}" appears directly in run: block of step "send message via cli"; move to env: map

Locations:

- `action.yml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all three script-injection findings in action.yml:
1. 'download binary' step (lines 44-45): Replaced `${{ env.VERSION }}` and `${{ env.BINARY_NAME }}` with plain shell variable references `${VERSION}` and `${BINARY_NAME}`. These values are already defined in the step's `env:` block, so no GitHub expression interpolation is needed in the `run:` block.
2. 'send message via cli' step (lines 49/62): Moved `${{ inputs.webhook_url }}` from the `run:` block into the `env:` block as `WEBHOOK_URL: ${{ inputs.webhook_url }}`, and updated the shell command to reference it as `"$WEBHOOK_URL"`. This prevents attacker-controlled webhook URL values from injecting shell metacharacters.

