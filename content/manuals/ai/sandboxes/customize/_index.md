---
title: Customizing sandboxes
linkTitle: Customize
description: Build reusable sandbox images, extend agents with tools and credentials, and define custom agents using templates and kits.
keywords: sandboxes, sbx, customize, templates, kits, mixins, custom agents
weight: 35
aliases:
  - /ai/sandboxes/agents/custom-environments/
---

{{< summary-bar feature_name="Docker Sandboxes sbx" >}}

Docker Sandboxes offers two ways to customize a sandbox beyond the built-in
defaults:

- [Templates](templates.md) — reusable sandbox images with tools, packages,
  and configuration baked in. Extend a base image with a Dockerfile, or
  save a running sandbox as a template.
- [Kits](kits.md) — declarative YAML artifacts that extend an agent with
  tools, credentials, network rules, and files at runtime, or define a new
  agent from scratch.

## When to use which

| Goal                                                      | Option                                                        |
| --------------------------------------------------------- | ------------------------------------------------------------- |
| Pre-install tools and packages into a reusable base image | [Template](templates.md)                                      |
| Capture a configured running sandbox for reuse            | [Saved template](templates.md#saving-a-sandbox-as-a-template) |
| Add a tool, credential, or config to agent runs via YAML  | [Kit (mixin)](kits.md)                                        |
| Define a new agent from scratch                           | [Kit (agent)](kits.md#defining-an-agent)                      |

Templates and kits can be used together. A template bakes heavy tools into
the image for fast sandbox startup; a kit layered on top adds per-run
credentials, config, or extra capabilities.

## Tutorials

- [Build your own agent kit](build-an-agent.md) — step-by-step walkthrough
  for packaging [Amp](https://ampcode.com/) as an agent kit.
