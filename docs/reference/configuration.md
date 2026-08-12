---
title: Configuration Options
description: Every .shippy.yaml key with its type, default and the guide section that explains it.
---

# Configuration Options

A lookup table for `.shippy.yaml`. For explanations and examples, follow the links into the
[Configuration guide](../guide/configuration).

## Top level

| Key | Type | Description |
| --- | --- | --- |
| `hosts` | map | One entry per deployment target. The key is the name you pass to `shippy deploy <name>`. |
| `commands` | list | Commands executed in the new release directory, before it goes live. |
| `rollback_commands` | list | Commands executed when `shippy rollback` switches releases. |
| `backup` | map | Settings for `shippy backup`. Can be overridden per host. |

## `hosts.<name>`

### Connection

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `hostname` | string | — | Server domain or IP. |
| `port` | int | `22` | SSH port. Top-level for convenience; other SSH settings live in `ssh_options`. |
| `remote_user` | string | — | SSH username. |
| `ssh_key` | string | auto-detected | Path to the SSH **private** key. Falls back to `~/.ssh/id_ed25519`, `~/.ssh/id_rsa`, `~/.ssh/id_ecdsa`. See [SSH Connections](../guide/ssh#ssh-key-detection). |
| `ssh_options` | map | — | SSH configuration options, see [below](#ssh-options). |

### Deployment

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `deploy_path` | string | — | Absolute path on the server. Shippy creates `current`, `releases/` and `shared/` beneath it. |
| `rsync_src` | string | — | Local source directory, usually `./`. |
| `keep_releases` | int | `5` | How many releases to keep after cleanup — these are the ones you can roll back to. |

### Files

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `shared` | list | — | Paths symlinked from `shared/` into every release. Trailing `/` marks a directory. See [Shared Files/Directories](../guide/configuration#shared-files-directories). |
| `exclude` | list | — | Additional gitignore-style exclude patterns, on top of `.gitignore` and the [default excludes](../guide/configuration#default-excludes). |
| `include` | list | — | Patterns to transfer even though `.gitignore` excludes them. |

## `ssh_options`

All values are strings. See [SSH Options](../guide/ssh#ssh-options) for details.

| Option | Default | Description |
| --- | --- | --- |
| `ConnectTimeout` | `30` | Connection timeout. Accepts seconds (`"30"`) or a duration (`"30s"`, `"5m"`, `"1h"`). |
| `ServerAliveInterval` | — | Interval between keepalive messages. Same formats as `ConnectTimeout`. |
| `ServerAliveCountMax` | `3` | Failed keepalives tolerated before disconnecting. |
| `Compression` | — | `"yes"` / `"true"` / `"no"` / `"false"`. Negotiated with the server. |
| `StrictHostKeyChecking` | — | `"yes"`, `"accept-new"` (recommended) or `"no"`. |
| `UserKnownHostsFile` | `~/.ssh/known_hosts` | Known hosts file, `~` is expanded. |

## `commands` and `rollback_commands`

| Key | Type | Description |
| --- | --- | --- |
| `name` | string | Description shown in the deployment output. |
| `run` | string | Command executed in the new release directory. |

```yaml
commands:
  - name: Clear TYPO3 cache
    run: ./{{config.bin-dir|vendor/bin}}/typo3 cache:flush
```

## `backup`

See [`shippy backup`](./commands#backup) for the full annotated block.

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `output` | string | cwd | Local directory the ZIP is written to. |
| `files` | list | — | Paths inside the remote `shared/` directory to include. |
| `database.credentials` | string | `auto` | `auto`, `dotenv`, `typo3` or `manual`. |
| `database.driver` | string | — | `mysql`, `postgresql` or `sqlite`. Required with `credentials: manual`. |
| `database.host` / `.port` / `.name` / `.user` / `.password` | string / int | — | Explicit credentials, used with `credentials: manual`. Supports `${VAR}` substitution. |
| `database.exclude_tables` | list | — | Glob patterns for tables to skip in the dump. |
| `database.options` | map | — | DBMS-specific options, e.g. `single_transaction`, `charset` (MySQL), `schema` (PostgreSQL). |

## Substitutions

Both forms work in any string value. See
[Template Variables](../guide/configuration#template-variables) and
[Environment Variables](../guide/configuration#environment-variables).

| Syntax | Source | Example |
| --- | --- | --- |
| `{{key.path}}`{v-pre} | `composer.json` | `{{name}}`{v-pre}, `{{extra.typo3/cms.web-dir}}`{v-pre} |
| `{{key.path\|fallback}}`{v-pre} | `composer.json`, with fallback | `{{config.bin-dir\|vendor/bin}}`{v-pre} |
| `${VAR}` | environment | `${DEPLOY_HOST}` |
| `${VAR\|fallback}` | environment, with fallback | `${DEPLOY_PATH\|/var/www/html}` |

::: warning
An unset `${VAR}` without a fallback fails the deployment.
:::
