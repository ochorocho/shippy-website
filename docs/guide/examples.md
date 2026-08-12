---
title: Example Configurations
description: A minimal TYPO3 configuration and an advanced multi-host setup with staging and production.
---

# Example Configurations

Two complete `.shippy.yaml` files to copy from. Every key used here is explained in
[Configuration](./configuration).

## Minimal TYPO3 Configuration

```yaml
hosts:
  production:
    hostname: www.example.com
    remote_user: deploy
    deploy_path: /var/www/{{name}}
    rsync_src: ./
    ssh_key: ~/.ssh/id_rsa
    shared:
      - .env
      - var/log/
      - var/session/
      - public/fileadmin/
      - public/uploads/

commands:
  - name: Clear TYPO3 cache
    run: ./vendor/bin/typo3 cache:flush

  - name: Run extension setup
    run: ./vendor/bin/typo3 extension:setup
```

## Advanced Configuration

Two hosts with different retention, per-host excludes, and a fuller command list. Note the
`{{config.bin-dir|vendor/bin}}`{v-pre} template variable, which reads the binary directory from
`composer.json` and falls back to `vendor/bin`.

```yaml
hosts:
  staging:
    hostname: staging.example.com
    remote_user: deploy
    deploy_path: /var/www/{{name}}/staging
    rsync_src: ./
    ssh_key: ~/.ssh/id_rsa
    keep_releases: 3

    # Additional excludes beyond .gitignore
    exclude:
      - .git/
      - node_modules/
      - .env.local
      - Tests/

    # Force include despite .gitignore
    include:
      - public/.htaccess

    # Shared paths
    shared:
      - .env
      - var/log/
      - var/session/
      - public/fileadmin/
      - public/uploads/

  production:
    hostname: www.example.com
    remote_user: deploy
    deploy_path: /var/www/{{name}}/production
    rsync_src: ./
    ssh_key: ~/.ssh/id_rsa_production
    keep_releases: 10
    shared:
      - .env
      - var/log/
      - var/session/
      - public/fileadmin/
      - public/uploads/

commands:
  - name: Clear TYPO3 cache
    run: ./{{config.bin-dir|vendor/bin}}/typo3 cache:flush

  - name: Run extension setup
    run: ./{{config.bin-dir|vendor/bin}}/typo3 extension:setup

  - name: Database migrations
    run: ./{{config.bin-dir|vendor/bin}}/typo3 upgrade:run

  - name: Warmup caches
    run: ./{{config.bin-dir|vendor/bin}}/typo3 cache:warmup

rollback_commands:
  - name: Flush caches
    run: ./{{config.bin-dir|vendor/bin}}/typo3 cache:flush

  - name: Warmup caches
    run: ./{{config.bin-dir|vendor/bin}}/typo3 cache:warmup
```

Deploy either host by name:

```bash
shippy deploy staging
shippy deploy production
```

## Backup configuration

Add a `backup:` block to have `shippy backup <host>` produce a ZIP with a database dump and the
shared files worth keeping:

```yaml
backup:
  output: ./backups            # Local directory for ZIPs (default: cwd)

  # Files to download from the remote shared/ directory (paths are relative to shared/)
  files:
    - .env
    - public/fileadmin/
    - public/uploads/

  database:
    credentials: auto

    # Exclude tables from the dump (glob patterns)
    exclude_tables:
      - "cache_*"
      - "cf_*"
      - "sys_log"
      - "be_sessions"

    # DBMS-specific options
    options:
      single_transaction: "true"   # MySQL: consistent dump without table locks
```

The full set of `backup:` options — including manual database credentials and per-host overrides — is
documented under [`backup`](../reference/commands#backup).
