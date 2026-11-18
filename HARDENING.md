<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--send-google-chat-webhook/v0.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--send-google-chat-webhook/v0.0.4** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'download binary' step puts ${{ env.VERSION }} and ${{ env.BINARY_NAME }} straight into the run: script. Offending lines: `curl -LOv "https://github.com/google-github-actions/send-google-chat-webhook/releases/download/v${{ env.VERSION }}/send-google-chat-webhook_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz"` and `tar xzf ${{ env.BINARY_NAME }}_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz`. Fix: reference the step's existing env vars as "${VERSION}" and "${BINARY_NAME}" in double quotes, and quote the whole tar filename.

Locations:

- `action.yml:57`
- `action.yml:58`

### script-injection (severity: high)

Rule (a): the 'send message via cli' step puts the caller-supplied input ${{ inputs.webhook_url }} straight into the run: script, so a crafted value can break out of the quotes and run commands. Offending line: `./send-google-chat-webhook chat workflownotification --webhook-url="${{ inputs.webhook_url }}"`. Fix: map it through env (e.g. WEBHOOK_URL: ${{ inputs.webhook_url }}) and use --webhook-url="${WEBHOOK_URL}".

Locations:

- `action.yml:62`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.webhook_url }}" appears directly in run: block of step "send message via cli"; move to env: map

Locations:

- `action.yml:62`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index ef443dd..4be0c7e 100644
--- a/action.yml
+++ b/action.yml
@@ -15,17 +15,33 @@
 name: 'send-google-chat-webhhook'
 description: "send message to your google chat workspace"
 inputs:
-  webhook_url: 
+  webhook_url:
     description: "chat space webhook url"
     required: true
   mention:
     description: "mention people or not, format <users/user_id>"
     default: '<users/all>'
     required: false
-
 runs:
   using: 'composite'
   steps:
     - name: 'download binary'
       shell: 'bash'
       env:
@@ -54,13 +70,14 @@ runs:
           ;;
         esac
 
-        curl -LOv "https://github.com/google-github-actions/send-google-chat-webhook/releases/download/v${{ env.VERSION }}/send-google-chat-webhook_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz"
-        tar xzf ${{ env.BINARY_NAME }}_${{ env.VERSION }}_${CURL_OS}_${CURL_ARCH}.tar.gz
+        curl -LOv "https://github.com/google-github-actions/send-google-chat-webhook/releases/download/v${VERSION}/send-google-chat-webhook_${VERSION}_${CURL_OS}_${CURL_ARCH}.tar.gz"
+        tar xzf ${BINARY_NAME}_${VERSION}_${CURL_OS}_${CURL_ARCH}.tar.gz
     - name: "send message via cli"
       shell: 'bash'
       run: |
-        ./send-google-chat-webhook chat workflownotification --webhook-url="${{ inputs.webhook_url }}"
+        ./send-google-chat-webhook chat workflownotification --webhook-url="$WEBHOOK_URL"
       env:
+        WEBHOOK_URL: ${{ inputs.webhook_url }}
         GITHUB_CONTEXT: ${{ toJson(github) }}
         JOB_CONTEXT: ${{ toJson(job) }}
         STEPS_CONTEXT: ${{ toJson(steps) }}

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

action.yml: the 'download binary' step now reads "${VERSION}" and "${BINARY_NAME}" from its existing env block, and the tar filename is now quoted. The 'send message via cli' step now passes inputs.webhook_url through env as WEBHOOK_URL and uses --webhook-url="${WEBHOOK_URL}". This puts back the security changes from the previous harden. I haven't run the action to test it.

