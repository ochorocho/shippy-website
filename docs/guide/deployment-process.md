---
title: Deployment Process
description: The eight steps Shippy performs on every deploy, and why the site only goes live at step seven.
---

# Deployment Process

When you run `shippy deploy <host>`, the following steps occur:

1. **Scan files** — Walks source directory, applies the deny-by-default allowlist (include/exclude
   patterns)
2. **Connect to server** — Establishes SSH connection
3. **Create release** — Creates new timestamped release directory (e.g. `releases/20260109203841`)
4. **Sync files** — Transfers files to the new release directory
5. **Create symlinks** — Links shared files/directories from `shared/` to the release
6. **Execute commands** — Runs commands **in the new release directory** (e.g. cache flush,
   migrations)
7. **Activate release** — Atomically updates `current` symlink to new release (site goes live)
8. **Cleanup** — Removes old releases, keeps last N

::: info Commands run before the release goes live
Commands execute in the new release directory **before** it goes live. This ensures all preparation
(cache warming, migrations, etc.) completes successfully before the atomic switchover. The site only
becomes live when the `current` symlink is updated in step 7.
:::

## Why this is zero-downtime

Steps 3 to 6 happen in a directory nobody is serving yet. Your visitors keep hitting the previous
release the entire time files are uploading and commands are running. Step 7 replaces a single
symlink, which is atomic at the filesystem level — there is no window in which the document root
points at a partially written release.

That also means a failure during steps 1–6 leaves production untouched: the broken release directory
is simply never activated.

## Previewing a deployment

`--dry-run` runs the scan and resolves the command list without connecting to the host, so you can
confirm the allowlist picks up what you expect before anything is transferred:

```bash
shippy deploy production --dry-run
```

## One deployment at a time

Shippy takes a lock on the host for the duration of a deploy, so a second deployment cannot start
while one is running. It is on by default and expires after 15 minutes — see
[Deployment Locking](./configuration#deployment-locking), and
[`shippy unlock`](../reference/commands#unlock) if a crashed run leaves a lock behind.

## Rolling back

Because the previous releases are still on disk (see `keep_releases` in
[Configuration](./configuration#directory-structure)), undoing a deployment is the same symlink
swap in reverse:

```bash
shippy rollback production              # Interactive release selection
shippy rollback production -n -1        # One version back
```

Any `rollback_commands` you configured — typically a cache flush and warmup — run as part of it. See
the [`rollback` reference](../reference/commands#rollback) for all flags.

## What Shippy does not do

Shippy deploys files and runs commands. It does not manage your web server, PHP-FPM pools, database
schema (beyond whatever your own commands do), or DNS. That is deliberate: the same eight steps
behave identically on any host you can reach over SSH.
