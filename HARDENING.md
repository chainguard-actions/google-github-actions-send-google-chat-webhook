<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--send-google-chat-webhook/v0.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **google-github-actions--send-google-chat-webhook/v0.0.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'download binary' step directly interpolates ${{ env.VERSION }} and ${{ env.BINARY_NAME }} inside the run: shell block. Even though these values are set in the same step's env: block, YAML template substitution occurs before the shell executes, making any ${{ ... }} inside a run: block a script-injection risk. Offending lines:
  curl -LOv "https://.../v${{ env.VERSION }}/send-google-chat-webhook_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz"
  tar xzf ${{ env.BINARY_NAME }}_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz
Additionally, sub-rule (b): the tar xzf line uses ${{ env.BINARY_NAME }} and ${{ env.VERSION }} without double-quoting, allowing shell metacharacter injection.

Locations:

- `action.yml:55`
- `action.yml:56`

### script-injection (severity: high)

Sub-rule (a): The 'send message via cli' step directly interpolates the attacker-controlled input ${{ inputs.webhook_url }} into the run: shell command string:
  ./send-google-chat-webhook chat workflownotification --webhook-url="${{ inputs.webhook_url }}"
An attacker can supply a malicious webhook_url value containing shell metacharacters (e.g. "; malicious-command #") that will be executed by the shell. The value should instead be passed via an env: variable and referenced as a quoted shell variable (e.g. "$WEBHOOK_URL").

Locations:

- `action.yml:61`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.webhook_url }}" appears directly in run: block of step "send message via cli"; move to env: map

Locations:

- `action.yml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all three script-injection findings in action.yml:
1. 'download binary' step (lines 55-56): Replaced ${{ env.VERSION }} and ${{ env.BINARY_NAME }} template expressions in the run: block with plain shell variable references ${VERSION} and ${BINARY_NAME}. These values are already defined in the step's env: block, so no YAML template interpolation is needed. Also added double-quoting around the tar filename argument to prevent shell metacharacter injection.
2. 'send message via cli' step (lines 61-62): Moved ${{ inputs.webhook_url }} out of the run: shell block into the step's env: block as WEBHOOK_URL, and updated the shell command to reference it as "$WEBHOOK_URL" instead of the inline template expression.

