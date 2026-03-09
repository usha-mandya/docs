---
title: Credentials
weight: 20
description: How Docker Sandboxes handle API keys and authentication credentials for sandboxed agents.
---

{{< summary-bar feature_name="Docker Sandboxes sbx" >}}

Most agents need an API key for their model provider. An HTTP/HTTPS proxy on
your host intercepts outbound API requests from the sandbox and injects the
appropriate authentication headers before forwarding each request. Your
credentials stay on the host and are never stored inside the sandbox VM. For
how this works as a security layer, see
[Credential isolation](isolation.md#credential-isolation).

There are two ways to provide credentials:

- **Stored secrets** (recommended): saved in your OS keychain, encrypted and
  persistent across sessions.
- **Environment variables:** read from your current shell session. This works
  but is less secure on the host side, since environment variables are visible
  to other processes running as your user.

If both are set for the same service, the stored secret takes precedence. For
multi-provider agents (OpenCode, Docker Agent), the proxy automatically selects the
correct credentials based on the API endpoint being called. See individual
[agent pages](../agents/) for provider-specific details.

## Stored secrets

The `sbx secret` command stores credentials in your OS keychain so you don't
need to export environment variables in every terminal session. When a sandbox
starts, the proxy looks up stored secrets and uses them to authenticate API
requests on behalf of the agent. The secret is never exposed directly to the
agent.

### Store a secret

```console
$ sbx secret set -g anthropic
```

This prompts you for the secret value interactively. The `-g` flag stores the
secret globally so it's available to all sandboxes. To scope a secret to a
specific sandbox instead:

```console
$ sbx secret set my-sandbox openai
```

> [!NOTE]
> A sandbox-scoped secret takes effect immediately, even if the sandbox is
> running. A global secret (`-g`) only applies when a sandbox is created. If
> you set or change a global secret while a sandbox is running, recreate the
> sandbox for the new value to take effect.

You can also pipe in a value for non-interactive use:

```console
$ echo "$ANTHROPIC_API_KEY" | sbx secret set -g anthropic
```

### Supported services

Each service name maps to a set of environment variables the proxy checks and
the API domains it authenticates requests to:

| Service     | Environment variables                        | API domains                         |
| ----------- | -------------------------------------------- | ----------------------------------- |
| `anthropic` | `ANTHROPIC_API_KEY`                          | `api.anthropic.com`                 |
| `aws`       | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | AWS Bedrock endpoints               |
| `github`    | `GH_TOKEN`, `GITHUB_TOKEN`                   | `api.github.com`, `github.com`      |
| `google`    | `GEMINI_API_KEY`, `GOOGLE_API_KEY`           | `generativelanguage.googleapis.com` |
| `groq`      | `GROQ_API_KEY`                               | `api.groq.com`                      |
| `mistral`   | `MISTRAL_API_KEY`                            | `api.mistral.ai`                    |
| `nebius`    | `NEBIUS_API_KEY`                             | `api.studio.nebius.ai`              |
| `openai`    | `OPENAI_API_KEY`                             | `api.openai.com`                    |
| `xai`       | `XAI_API_KEY`                                | `api.x.ai`                          |

When you store a secret with `sbx secret set -g <service>`, the proxy uses it
the same way it would use the corresponding environment variable. You don't
need to set both.

### List and remove secrets

List all stored secrets:

```console
$ sbx secret ls
SCOPE      SERVICE   SECRET
(global)   github    gho_GCaw4o****...****43qy
```

Remove a secret:

```console
$ sbx secret rm -g github
```

> [!NOTE]
> Running `sbx reset` deletes all stored secrets along with all sandbox state.
> You'll need to re-add your secrets after a reset.

### GitHub token

The `github` service gives the agent access to the `gh` CLI inside the
sandbox. Pass your existing GitHub CLI token:

```console
$ echo "$(gh auth token)" | sbx secret set -g github
```

This is useful for agents that create pull requests, open issues, or interact
with GitHub APIs on your behalf.

## Environment variables

As an alternative to stored secrets, export the relevant environment variable
in your shell before running a sandbox:

```console
$ export ANTHROPIC_API_KEY=sk-ant-api03-xxxxx
$ sbx run claude
```

The proxy reads the variable from your terminal session. See individual
[agent pages](../agents/) for the variable names each agent expects.

## Best practices

- Use [stored secrets](#stored-secrets) over environment variables. The OS
  keychain encrypts credentials at rest and controls access, while environment
  variables are plaintext in your shell.
- Don't set API keys manually inside the sandbox. Credentials stored in
  environment variables or configuration files inside the VM are readable by
  the agent process directly.
- For Claude Code, the interactive OAuth flow is another secure option: the
  proxy handles authentication without exposing the token inside the sandbox.
  Leave `ANTHROPIC_API_KEY` unset to use OAuth.

## Custom templates and placeholder values

When building custom templates or installing agents manually in a shell
sandbox, some agents require environment variables like `OPENAI_API_KEY` to be
set before they start. Set these to placeholder values (e.g. `proxy-managed`)
if needed. The proxy injects actual credentials regardless of the environment
variable value.
