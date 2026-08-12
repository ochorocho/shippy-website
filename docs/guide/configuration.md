---
title: Configuration
description: Everything .shippy.yaml can express — hosts, excludes, template variables, shared paths and the on-server directory structure.
---

# Configuration

Shippy is configured by a single `.shippy.yaml` in your project root. `shippy init` generates one for
you; this page explains every part of it. For a terse key-by-key table, see the
[Configuration Options reference](../reference/configuration).

## Basic Structure

```yaml
hosts:
  <hostname>:
    # SSH connection
    hostname: <server domain or IP>
    port: <SSH port, default: 22>
    remote_user: <SSH username>
    ssh_key: <path to SSH private key>
    ssh_options: <map of SSH options, see below>

    # Deployment
    deploy_path: <absolute path on server>
    rsync_src: <local source directory>
    keep_releases: <number of releases to keep, default: 5>

    # File management
    shared: <list of shared paths>
    exclude: <additional exclude patterns>
    include: <patterns to include despite .gitignore>

commands:
  - name: <command description>
    run: <command to execute>
```

The key under `hosts:` is the name you pass on the command line — `shippy deploy production` uses the
`hosts.production` entry.

## Exclude and Include Patterns

### Exclude Patterns

Shippy automatically respects `.gitignore` patterns **including nested `.gitignore` files in
subdirectories**. You can add additional exclude patterns in your configuration:

```yaml
hosts:
  production:
    hostname: example.com
    remote_user: deploy
    deploy_path: /var/www/myproject
    rsync_src: ./

    # Additional exclude patterns (beyond .gitignore)
    exclude:
      - "*.log"              # Exclude all .log files
      - ".env.example"       # Exclude specific file
      - "tests/"             # Exclude entire directory
      - "*.md"               # Exclude all markdown files
      - ".ddev/"             # Exclude DDEV configuration
      - "node_modules/"      # Exclude node modules (if not in .gitignore)
```

### Include Patterns

Use `include` to explicitly include files that are excluded by `.gitignore`:

```yaml
hosts:
  production:
    hostname: example.com
    remote_user: deploy
    deploy_path: /var/www/myproject
    rsync_src: ./

    # Force include files despite .gitignore
    include:
      - "public/.htaccess"   # Include .htaccess files
      - "vendor/"            # Include vendor directory (if gitignored)
      - ".env.production"    # Include specific environment file
```

::: tip
`include` is what makes a build-artifact deployment work: keep `vendor/` and `public/_assets/` out of
git, build them in CI, and list them under `include` so they still reach the server.
:::

### Pattern Syntax

- Patterns use gitignore-style syntax
- `*` matches any characters except `/`
- `**` matches any characters including `/`
- Trailing `/` means directory only
- No leading `/` means pattern matches at any depth
- Leading `/` means pattern matches from project root

**Examples:**

```yaml
exclude:
  - "*.log"                    # All .log files at any depth
  - "/build/"                  # build/ directory at root only
  - "temp/"                    # temp/ directory at any depth
  - "**/*.test.js"             # All .test.js files anywhere
  - ".DS_Store"                # macOS metadata files
  - "Thumbs.db"                # Windows metadata files
```

## Template Variables

Use `{{key.path}}`{v-pre} syntax to reference values from composer.json:

```yaml
hosts:
  production:
    deploy_path: /var/www/{{name}}  # Uses composer.json "name" field
```

Access nested values:

```yaml
deploy_path: /var/www/{{extra.typo3/cms.web-dir}}
```

Provide a fallback value with `|` (used when the key is not found in composer.json):

```yaml
commands:
  - name: Clear cache
    run: ./{{config.bin-dir|vendor/bin}}/typo3 cache:flush
```

Run [`shippy config validate`](../reference/commands#validate-configuration) to see what these
expand to before deploying.

## Environment Variables

Use `${VAR}` syntax to reference environment variables:

```yaml
hosts:
  production:
    hostname: ${DEPLOY_HOST}
    remote_user: ${DEPLOY_USER}
```

Provide a fallback value with `|` (used when the variable is not set):

```yaml
hosts:
  production:
    deploy_path: ${DEPLOY_PATH|/var/www/html}
```

::: warning
If an environment variable is not set and no fallback is provided, deployment will fail with an
error.
:::

This is how you keep secrets and per-environment values out of the repository — see
[CI/CD](./ci-cd).

## Shared Files/Directories

Files and directories in the `shared:` list are symlinked from the `shared/` directory to each
release:

```yaml
shared:
  - .env                    # Shared file
  - var/log/                # Shared directory (note trailing slash)
  - public/fileadmin/
  - public/uploads/
```

Anything that must survive a deployment — user uploads, logs, sessions, the environment file — belongs
here. Everything else is replaced wholesale with each release.

## Directory Structure

Shippy creates the following structure on the server (following Deployer/Capistrano conventions):

```
/var/www/myproject/
├── current -> releases/20240109120000    # Symlink to latest release
├── releases/
│   ├── 20240109120000/                   # Current release
│   ├── 20240109110000/                   # Previous release
│   └── 20240109100000/                   # Older release
└── shared/
    ├── .env                              # Shared files
    ├── var/
    │   ├── log/
    │   └── session/
    └── public/
        ├── fileadmin/
        └── uploads/
```

Point your web server's document root at `current/public` (or wherever your web directory sits inside
the project). Because `current` is a symlink that Shippy swaps atomically, no request ever sees a
half-written release.

`keep_releases` controls how many entries under `releases/` are retained after a successful deploy —
it defaults to `5`, and those are exactly the releases you can roll back to.

## Default Excludes

Shippy automatically excludes these patterns (in addition to .gitignore):

- `.git/`
- `.gitignore`
- `.shippy.yaml`
- `.shippy.yaml.example`
- `node_modules/`
- `.env.local`
- `.env.*.local`
- `var/cache/`
- `var/log/`
- `var/transient/`
- `.DS_Store`
- `Thumbs.db`

## Next steps

- [SSH Connections](./ssh) — keys, ports, timeouts and host key verification
- [Example Configurations](./examples) — complete minimal and advanced files
- [Configuration Options](../reference/configuration) — every key in one table
