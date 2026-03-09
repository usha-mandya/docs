---
title: Codex
weight: 20
description: |
  Use OpenAI Codex in Docker Sandboxes with API key authentication and YOLO
  mode configuration.
---

{{< summary-bar feature_name="Docker Sandboxes sbx" >}}

This guide covers authentication, configuration, and usage of Codex in a
sandboxed environment.

Official documentation: [Codex CLI](https://developers.openai.com/codex/cli)

## Quick start

Create a sandbox and run Codex for a project directory:

```console
$ sbx run codex ~/my-project
```

The workspace parameter is optional and defaults to the current directory:

```console
$ cd ~/my-project
$ sbx run codex
```

## Authentication

Codex requires an OpenAI API key. Store your key using
[stored secrets](../security/credentials.md#stored-secrets):

```console
$ sbx secret set -g openai
```

Alternatively, export the `OPENAI_API_KEY` environment variable in your shell
before running the sandbox. See
[Credentials](../security/credentials.md) for details on both methods.

## Configuration

Sandboxes don't pick up user-level configuration from your host, such as
`~/.codex`. Only project-level configuration in the working directory is
available inside the sandbox. See
[Why doesn't the sandbox use my user-level agent configuration?](../faq.md#why-doesnt-the-sandbox-use-my-user-level-agent-configuration)
for workarounds.

The sandbox runs Codex without approval prompts by default. Pass additional
Codex CLI options after `--`:

```console
$ sbx run codex --name <sandbox-name> -- <codex-options>
```

## Base image

Template: `docker/sandbox-templates:codex`

Preconfigured to run without approval prompts.

See [Custom environments](custom-environments.md) to pre-install tools or
customize this environment.
