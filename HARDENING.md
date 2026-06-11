<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--send-google-chat-webhook/v0.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **google-github-actions--send-google-chat-webhook/v0.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: ${{ env.VERSION }} and ${{ env.BINARY_NAME }} are interpolated directly inside a run: shell command in the 'download binary' step. The env.* context is workflow-controllable and flows through YAML template substitution before the shell processes it, enabling script injection. Offending lines:
  curl -LOv "https://...v${{ env.VERSION }}/send-google-chat-webhook_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz"
  tar xzf ${{ env.BINARY_NAME }}_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz
Fix: move VERSION and BINARY_NAME into the env: block (they already are) and reference them as $VERSION / $BINARY_NAME shell variables instead of ${{ }} expressions.

Locations:

- `action.yml:55`

### script-injection (severity: high)

Rule (a) violation: ${{ inputs.webhook_url }} is interpolated directly inside a run: shell command in the 'send message via cli' step. inputs.webhook_url is attacker-controlled and is injected into the shell command string before the shell parses it, enabling arbitrary command injection. Offending line:
  ./send-google-chat-webhook chat workflownotification --webhook-url="${{ inputs.webhook_url }}"
Fix: pass the value via an env: variable (e.g. WEBHOOK_URL: ${{ inputs.webhook_url }}) and reference it as "$WEBHOOK_URL" in the run: script.

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

Fixed three script injection findings in action.yml:
1. 'download binary' step: Replaced ${{ env.VERSION }} and ${{ env.BINARY_NAME }} template expressions in the run: block with shell variable references ${VERSION} and ${BINARY_NAME}. These variables were already declared in the step's env: block, so no new env entries were needed.
2. 'send message via cli' step: Moved ${{ inputs.webhook_url }} out of the run: block and into the env: block as WEBHOOK_URL: ${{ inputs.webhook_url }}, then updated the shell command to reference it as "$WEBHOOK_URL" instead of the direct expression interpolation.

