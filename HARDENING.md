<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--send-google-chat-webhook/v0.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--send-google-chat-webhook/v0.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ }}` expressions are interpolated directly inside `run:` shell command strings in the 'download binary' step. `${{ env.VERSION }}` and `${{ env.BINARY_NAME }}` are substituted by the Actions template engine before the shell ever sees the string, meaning any value that reaches these expressions is injected raw into the shell. Offending lines:
  - `curl -LOv "https://...v${{ env.VERSION }}/send-google-chat-webhook_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz"` (line 54)
  - `tar xzf ${{ env.BINARY_NAME }}_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz` (line 55 — also unquoted)

Locations:

- `action.yml:54`
- `action.yml:55`

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.webhook_url }}` is interpolated directly inside the `run:` shell command string in the 'send message via cli' step. `inputs.webhook_url` is attacker-controlled (it is a required input supplied by the calling workflow) and is injected raw into the shell before quoting can take effect. Offending line:
  - `./send-google-chat-webhook chat workflownotification --webhook-url="${{ inputs.webhook_url }}"`

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

Fixed all script injection findings in hardened/action/action.yml:
1. 'download binary' step (lines 54-55): Replaced ${{ env.VERSION }} and ${{ env.BINARY_NAME }} template expressions with plain shell variable references ${VERSION} and ${BINARY_NAME}. These values are already defined in the step's env: block, so no Actions template interpolation is needed. Also properly quoted the tar argument.
2. 'send message via cli' step (lines 58/62): Moved ${{ inputs.webhook_url }} from the run: shell string into the step's env: block as WEBHOOK_URL. The shell command now uses "$WEBHOOK_URL" instead of the raw template expression, preventing attacker-controlled input from being injected into the shell.

