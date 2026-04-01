---
title: Policies
weight: 30
description: Configure network access and filesystem rules for sandboxes.
---

{{< summary-bar feature_name="Docker Sandboxes sbx" >}}

Sandboxes are [network-isolated](isolation.md) from your host and from each
other. A policy system controls what a sandbox can access — which external
hosts it can reach over the network, and which host paths it can mount as
workspaces.

Policies can be set at two levels:

- **Organization policies** {{< badge color=blue text="Limited Access" >}} — configured by admins in the
  [Docker Admin Console](https://app.docker.com/admin) under AI governance
  settings. These apply to all sandboxes across the organization.
- **Local policies** — configured by individual users with the `sbx policy`
  command. These apply to all sandboxes on the local machine.

If your organization has enabled governance, organization policies replace
local rules — local `sbx policy` rules are not evaluated unless an admin
also turns on the **User defined** setting. See [Precedence](#precedence)
for details.

## Organization policies {tier="Limited Access"}

> [!NOTE]
> Organization governance is a Limited Access feature. Contact your Docker
> account team to request access.

Organization admins can centrally manage policies through the
[Docker Admin Console](https://app.docker.com/admin). Navigate to your
organization settings and enable **Manage AI governance**.

Once enabled, the policies defined in the Admin Console apply to all
sandboxes across the organization.

### Local extensions to organization policy

When organization governance is active, local rules are ignored by default.
Admins can optionally let users extend the organization policy by turning on
the **User defined** setting in AI governance settings. When turned on,
local `sbx policy` rules are evaluated alongside organization rules, letting
users add hosts to the allowlist from their own machine using
`sbx policy allow network`.

Local extensions can expand access for domains the organization hasn't
explicitly denied, but can't override organization-level deny rules. This
applies to exact matches and wildcard matches alike — if the organization
denies `*.example.com`, a local allow for `api.example.com` has no effect
because the org-level wildcard deny covers it.

For example, given an organization policy that allows `api.anthropic.com`
and denies `*.corp.internal`:

- `sbx policy allow network api.example.com` — works, because the
  organization hasn't denied `api.example.com`
- `sbx policy allow network build.corp.internal` — no effect, because the
  organization denies `*.corp.internal`

## Network policies

The only way traffic can leave a sandbox is through an HTTP/HTTPS proxy on
your host, which enforces access rules on every outbound request.

### Initial policy selection

On first start, and after running `sbx policy reset`, the daemon prompts you to
choose a network policy:

```plaintext
Choose a default network policy:

     1. Open         — All network traffic allowed, no restrictions.
     2. Balanced     — Default deny, with common dev sites allowed.
     3. Locked Down  — All network traffic blocked unless you allow it.

  Use ↑/↓ to navigate, Enter to select, or press 1–3.
```

| Policy      | Description                                                                                                                                                                                    |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Open        | All outbound traffic is allowed. No restrictions. Equivalent to adding a wildcard allow rule with `sbx policy allow network "**"`.                                                             |
| Balanced    | Default deny, with a baseline allowlist covering AI provider APIs, package managers, code hosts, container registries, and common cloud services. You can extend this with `sbx policy allow`. |
| Locked Down | All outbound traffic is blocked, including model provider APIs (for example, `api.anthropic.com`). You must explicitly allow everything you need.                                              |

You can change your effective policy at any time using `sbx policy allow` and
`sbx policy deny`, or start over by running `sbx policy reset`.

> [!NOTE]
> If your organization manages AI governance policies, organization rules
> take precedence over the policy you select here. See
> [Organization policies](#organization-policies).

### Non-interactive environments

In non-interactive environments such as CI pipelines or headless servers, the
interactive prompt can't be displayed. Use `sbx policy set-default` to set the
default network policy before running any other `sbx` commands:

```console
$ sbx policy set-default balanced
```

Available values are `allow-all`, `balanced`, and `deny-all`. After setting the
default, you can customize further with `sbx policy allow` and
`sbx policy deny` as usual.

### Default policy

All outbound HTTP/HTTPS traffic is blocked by default unless an explicit rule
allows it. The **Balanced** policy ships with a baseline allowlist covering AI provider
APIs, package managers, code hosts, container registries, and common cloud
services. Run `sbx policy ls` to see the active rules for your installation.

Organization admins can modify or remove these defaults when configuring
[organization policies](#organization-policies).

### Managing local rules

Use [`sbx policy allow`](/reference/cli/sbx/policy/allow/) and
[`sbx policy deny`](/reference/cli/sbx/policy/deny/) to add network access
rules. Changes take effect immediately and apply to all sandboxes:

```console
$ sbx policy allow network api.anthropic.com
$ sbx policy deny network ads.example.com
```

Specify multiple hosts in one command with a comma-separated list:

```console
$ sbx policy allow network "api.anthropic.com,*.npmjs.org,*.pypi.org"
```

List all active policy rules with `sbx policy ls`:

```console
$ sbx policy ls
ID                                     TYPE      DECISION   RESOURCES
a1b2c3d4-e5f6-7890-abcd-ef1234567890   network   allow      api.anthropic.com, *.npmjs.org
f9e8d7c6-b5a4-3210-fedc-ba0987654321   network   deny       ads.example.com
```

Use `--type network` to show only network policies.

Remove a policy by resource or by rule ID:

```console
$ sbx policy rm network --resource ads.example.com
$ sbx policy rm network --id 2d3c1f0e-4a73-4e05-bc9d-f2f9a4b50d67
```

### Resetting to defaults

To remove all custom policies and restore the default policy, use
`sbx policy reset`:

```console
$ sbx policy reset
```

This deletes the local policy store and stops the daemon. When the daemon
restarts on the next command, you are prompted to choose a new network policy.
If sandboxes are running, they stop when the daemon shuts down. You are prompted for
confirmation unless you pass `--force`:

```console
$ sbx policy reset --force
```

### Switching to allow-by-default

If you prefer a permissive policy where all outbound traffic is allowed, add
a wildcard allow rule:

```console
$ sbx policy allow network "**"
```

This lets agents install packages and call any external API without additional
configuration. You can still deny specific hosts with `sbx policy deny`.

### Org-level rules {tier="Limited Access"}

Define network allow and deny rules in the Admin Console under
**AI governance > Network access**. Each rule takes a network target (domain,
wildcard, or CIDR range) and an action (allow or deny). You can add multiple
entries at once, one per line.

Organization-level rules use the same [wildcard syntax](#wildcard-syntax) as
local rules.

### Wildcard syntax

Rules support exact domains (`example.com`), wildcard subdomains
(`*.example.com`), and optional port suffixes (`example.com:443`).

Note that `example.com` doesn't match subdomains, and `*.example.com` doesn't
match the root domain. Specify both to cover both.

### Common patterns

Allow access to package managers so agents can install dependencies:

```console
$ sbx policy allow network "*.npmjs.org,*.pypi.org,files.pythonhosted.org,github.com"
```

The **Balanced** policy already includes AI provider APIs, package managers,
code hosts, and container registries. You only need to add allow rules for
additional domains your workflow requires. If you chose **Locked Down**, you
must explicitly allow everything.

> [!WARNING]
> Allowing broad domains like `github.com` permits access to any content on
> that domain, including user-generated content. Only allow domains you trust
> with your data.

### Monitoring

Use `sbx policy log` to see which hosts your sandboxes have contacted:

```console
$ sbx policy log
Blocked requests:
SANDBOX      TYPE     HOST                   PROXY        RULE       LAST SEEN        COUNT
my-sandbox   network  blocked.example.com    transparent  policykit  10:15:25 29-Jan  1

Allowed requests:
SANDBOX      TYPE     HOST                   PROXY        RULE       LAST SEEN        COUNT
my-sandbox   network  api.anthropic.com      forward      policykit  10:15:23 29-Jan  42
my-sandbox   network  registry.npmjs.org     transparent  policykit  10:15:20 29-Jan  18
```

The **PROXY** column shows how the request left the sandbox:

| Value         | Description                                                                                         |
| ------------- | --------------------------------------------------------------------------------------------------- |
| `forward`     | Routed through the forward proxy. Supports [credential injection](credentials.md).                  |
| `transparent` | Intercepted by the transparent proxy. Policy is enforced but credential injection is not available. |
| `network`     | Non-HTTP traffic (raw TCP, UDP, ICMP). Always blocked.                                              |

Filter by sandbox name by passing it as an argument:

```console
$ sbx policy log my-sandbox
```

Use `--limit N` to show only the last `N` entries, `--json` for
machine-readable output, or `--type network` to filter by policy type.

## Filesystem policies

Filesystem policies control which host paths a sandbox can mount as
workspaces. By default, sandboxes can mount any directory the user has
access to.

### Org-level restrictions {tier="Limited Access"}

Admins can restrict which paths are mountable by defining filesystem allow
and deny rules in the Admin Console under **AI governance > Filesystem access**.
Each rule takes a path pattern and an action (allow or deny).

> [!CAUTION]
> Use `**` (double wildcard) rather than `*` (single wildcard) when writing
> path patterns to match path segments recursively. A single `*` only matches
> within a single path segment. For example, `~/**` matches all paths under the
> user's home directory, whereas `~/*` matches only paths directly under `~`.

## Precedence

Within any layer, deny rules beat allow rules — if a domain matches both,
it's blocked regardless of specificity.

All outbound traffic is blocked by default unless an explicit rule allows it.
How rules are evaluated depends on whether organization governance is active.

Without organization governance, local rules (`sbx policy`) are the only
rules evaluated against this default-deny baseline.

With organization governance, local rules are not evaluated. Only
organization rules (Docker Admin Console) determine what is allowed or
denied. Organization-level denials can't be overridden locally.

If the admin turns on the **User defined** setting, local rules are also
evaluated alongside organization rules. Local rules can expand access for
domains the organization hasn't explicitly denied, but can't override
organization-level denials.

The same model applies to filesystem policies: organization-level rules take
precedence over local behavior.

To unblock a domain, identify where the deny rule comes from. For local rules,
remove it with `sbx policy rm`. For organization-level rules, contact your
organization admin.

## Troubleshooting

### Policy changes not taking effect

If policy changes aren't taking effect, run `sbx policy reset` to wipe the
local policy store and restart the daemon. On next start, you are prompted to
choose a new network policy, and the latest organization policies are pulled if
governance is enabled.

### Sandbox cannot mount workspace

If a sandbox fails to mount with a `mount policy denied` error, verify that
the filesystem allow rule uses `**` rather than `*`. A single `*` doesn't
match across directory separators.
