<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--send-google-chat-webhook/v0.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--send-google-chat-webhook/v0.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'download binary' step in action.yml interpolates ${{ env.VERSION }} and ${{ env.BINARY_NAME }} directly inside run: shell commands. Although these values come from the step's own env: block, any ${{ ... }} expression inside a run: block undergoes YAML template substitution before the shell processes it, enabling script injection if the values contain shell metacharacters. Offending lines:
  curl -LOv "https://.../v${{ env.VERSION }}/send-google-chat-webhook_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz"
  tar xzf ${{ env.BINARY_NAME }}_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz
Fix: replace ${{ env.VERSION }} with the shell env var $VERSION and ${{ env.BINARY_NAME }} with $BINARY_NAME (already set in the env: block), so no expression interpolation occurs inside the run: script.

Locations:

- `action.yml:55`
- `action.yml:56`

### script-injection (severity: high)

Sub-rule (a): The 'send message via cli' step in action.yml interpolates the attacker-controlled input ${{ inputs.webhook_url }} directly inside a run: shell command. This allows an attacker who controls the webhook_url input to inject arbitrary shell commands. Offending line:
  ./send-google-chat-webhook chat workflownotification --webhook-url="${{ inputs.webhook_url }}"
Fix: move the value to an env: variable (e.g. WEBHOOK_URL: ${{ inputs.webhook_url }}) and reference it as "$WEBHOOK_URL" inside the run: script.

Locations:

- `action.yml:60`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.webhook_url }}" appears directly in run: block of step "send message via cli"; move to env: map

Locations:

- `action.yml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed three script-injection findings in hardened/action/action.yml:
1. 'download binary' step (lines 55-56): Replaced `${{ env.VERSION }}` with `${VERSION}` and `${{ env.BINARY_NAME }}` with `${BINARY_NAME}`. These values are already defined in the step's env: block, so plain shell variable references are safe and avoid YAML template substitution.
2. 'send message via cli' step (lines 60/62): Moved `${{ inputs.webhook_url }}` from the run: shell command into the env: block as `WEBHOOK_URL: ${{ inputs.webhook_url }}`, and updated the shell command to reference `"$WEBHOOK_URL"` instead. This prevents attacker-controlled webhook_url input from being injected directly into the shell command string.

