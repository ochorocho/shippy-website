---
title: Commands
description: Every Shippy command — init, deploy, rollback, backup, gitlab:upload and config validate — with its flags.
---

# Commands

## Overview

| Command | Purpose |
| --- | --- |
| [`shippy init`](#initialize) | Create a `.shippy.yaml` with TYPO3 defaults |
| [`shippy deploy <host>`](#deploy) | Deploy to a target host |
| [`shippy rollback <host>`](#rollback) | Switch `current` to another release |
| [`shippy backup <host>`](#backup) | Download a database dump and shared files as a ZIP |
| [`shippy gitlab:upload <file>`](#upload-to-gitlab) | Push a file to the GitLab package registry |
| [`shippy config validate`](#validate-configuration) | Check the configuration file |

## Initialize

Create a new configuration file with TYPO3 defaults:

```bash
shippy init
```

Options:

- `--force` or `-f` — Overwrite existing configuration file

This command:

- Checks for `composer.json` in current directory
- Reads project name from composer.json
- Generates `.shippy.yaml` with sensible TYPO3 defaults
- Protects against accidental overwrites (use `--force` to override)

## Deploy

Deploy to a target host:

```bash
shippy deploy <hostname>
```

Example:

```bash
shippy deploy staging
shippy deploy production
```

`<hostname>` is the key under `hosts:` in `.shippy.yaml`, not the server's domain name. The eight
steps this runs through are described in
[Deployment Process](../guide/deployment-process).

## Rollback

Rollback to a previous release:

```bash
shippy rollback <hostname>
```

Options:

- `--list` or `-l` — List available releases and exit
- `--release` or `-r` — Switch to a specific release by name
- `--offset` or `-n` — Relative offset from current release (negative = older, positive = newer)

Examples:

```bash
shippy rollback production              # Interactive release selection
shippy rollback production -l           # List available releases
shippy rollback production -n -1        # One version back
shippy rollback production -n +1        # One version forward (e.g., after accidental rollback)
shippy rollback production -n -2        # Two versions back
shippy rollback production -r 20260109120000  # Specific release by name
```

When run without flags, shows an interactive list of available releases with deployment date/time,
git commit hash, and git tag. The current release is marked and cannot be selected.

## Backup

Create a ZIP archive containing a database dump and selected files from the remote `shared/`
directory:

```bash
shippy backup <hostname>
```

The output file is named `backup-<hostname>-<timestamp>.zip` and is written to the configured
`output:` directory (default: current working directory).

Configuration in `.shippy.yaml`:

```yaml
backup:
  output: ./backups            # Local directory for ZIPs (default: cwd)

  # Files to download from the remote shared/ directory (paths are relative to shared/)
  files:
    - .env
    - public/fileadmin/
    - public/uploads/

  database:
    # How database credentials are obtained:
    #   auto    - try `typo3 configuration:show` (TYPO3 v14+), then standard .env,
    #             then TYPO3 .env keys, then TYPO3 settings.php (default)
    #   dotenv  - read DB_HOST, DB_DATABASE, DB_USERNAME, DB_PASSWORD, ... from .env
    #   typo3   - try `typo3 configuration:show` (TYPO3 v14+), then TYPO3-specific
    #             .env keys or settings.php
    #   manual  - use the explicit driver/host/port/name/user/password fields below
    credentials: auto

    # Required only when credentials: manual
    # driver: mysql              # mysql | postgresql | sqlite
    # host: 127.0.0.1
    # port: 3306
    # name: my_database
    # user: db_user
    # password: ${DB_PASSWORD}   # environment-variable substitution is supported

    # Exclude tables from the dump (glob patterns)
    exclude_tables:
      - "cache_*"
      - "cf_*"
      - "sys_log"
      - "be_sessions"

    # DBMS-specific options
    options:
      single_transaction: "true"   # MySQL: consistent dump without table locks
      # charset: "utf8mb4"         # MySQL
      # schema: "public"           # PostgreSQL
```

With `credentials: auto` or `typo3`, Shippy first runs
`./vendor/bin/typo3 configuration:show DB/Connections/Default` (available in TYPO3 v14+) to read the
authoritative active database configuration. If the command is unavailable — older TYPO3, a
non-bootstrappable app, or a non-TYPO3 project — it falls back to parsing `.env` keys and
`config/system/settings.php` / legacy `typo3conf/LocalConfiguration.php`.

::: tip Per-host overrides
Add a `backup:` block inside a `hosts.<name>:` entry to override the global settings — useful when
staging and production need different exclude tables or output directories.
:::

## Upload to GitLab

Upload any file (typically a backup ZIP) to the GitLab Generic Packages registry of the project's git
origin:

```bash
shippy gitlab:upload <file>
```

The GitLab host and project path are auto-detected from the `origin` remote URL — no YAML config
required. Authentication resolves in this order:

1. `--token <token>` flag
2. `GITLAB_TOKEN` environment variable
3. `CI_JOB_TOKEN` environment variable (set automatically inside GitLab CI jobs)

Options:

- `--token` / `-t` — GitLab token
- `--package-name` — package name (default: project name from the git remote)
- `--package-version` — package version (default: timestamp, e.g. `20260501T102420`)

Typical local chain — back up production, then upload the archive:

```bash
shippy backup production
shippy gitlab:upload backups/backup-production-20260501T102420.zip --token "$GITLAB_TOKEN"
```

Run as a scheduled GitLab CI pipeline (uses `CI_JOB_TOKEN` automatically):

```yaml
# .gitlab-ci.yml
nightly_backup:
  stage: backup
  image: ghcr.io/ochorocho/shippy:latest
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
  script:
    - shippy backup production
    - shippy gitlab:upload backups/backup-production-*.zip
```

Uploaded archives appear in **Project → Deploy → Package Registry**, grouped by
`<package-name>/<package-version>`.

## Validate Configuration

Check if your configuration is valid:

```bash
shippy config validate
```

This command:

- Validates YAML syntax
- Checks required fields
- Tests composer.json template variables
- Shows processed configuration
