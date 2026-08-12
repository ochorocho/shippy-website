---
layout: home

hero:
  name: Shippy
  text: Deployments for PHP, in a single binary
  tagline: Zero-downtime, atomic releases for Composer-based projects. Inspired by Deployer and Capistrano — without the runtime.
  image:
    src: /logo.svg
    alt: Shippy
  actions:
    - theme: brand
      text: Get started
      link: /guide/quick-start
    - theme: alt
      text: What is Shippy?
      link: /guide/
    - theme: alt
      text: View on GitHub
      link: https://github.com/ochorocho/shippy

features:
  - icon: ⚓
    title: Zero-downtime releases
    details: Files are synced into a fresh release directory while the old one keeps serving. Going live is one atomic symlink swap.
    link: /guide/deployment-process
    linkText: How a deploy runs
  - icon: ↩️
    title: Rollback in seconds
    details: The last N releases stay on disk. Pick one from an interactive list, or step back with a single offset flag.
    link: /reference/commands#rollback
    linkText: rollback reference
  - icon: 🔗
    title: Shared files survive
    details: Uploads, logs, sessions and .env are symlinked from shared/ into every release, so nothing is lost between deployments.
    link: /guide/configuration#shared-files-directories
    linkText: Shared paths
  - icon: 📦
    title: One static binary
    details: Written in Go. Nothing to install on the server beyond SSH access, and nothing added to your composer.json.
    link: /guide/installation
    linkText: Install it
  - icon: 🧹
    title: Knows what to skip
    details: Respects .gitignore — including nested ones — plus your own exclude and include patterns, with gitignore-style syntax.
    link: /guide/configuration#exclude-and-include-patterns
    linkText: Patterns
  - icon: 🧡
    title: TYPO3 out of the box
    details: shippy init writes a config that already knows var/log/, public/fileadmin/ and the TYPO3 console commands.
    link: /guide/quick-start
    linkText: Quick start
---

## Install and ship

Three commands to install, four to deploy.

::: code-group

```bash [Homebrew]
brew tap ochorocho/shippy https://github.com/ochorocho/shippy
brew trust ochorocho/shippy
brew install shippy
```

```bash [Go]
go install github.com/ochorocho/shippy@latest
```

```bash [Docker]
docker run --rm ghcr.io/ochorocho/shippy:latest shippy --help
```

:::

```bash
shippy init                 # write .shippy.yaml from your composer.json
vim .shippy.yaml            # add hostname, remote_user, ssh_key
shippy config validate      # check syntax, fields and template variables
shippy deploy production    # ship it
```

## One file describes the whole deployment

No scripting DSL, no plugins — a host, the paths that must survive, and the commands to run before the
release goes live.

```yaml
hosts:
  production:
    hostname: www.example.com
    remote_user: deploy
    deploy_path: /var/www/{{name}}   # {{name}} comes from composer.json
    rsync_src: ./
    ssh_key: ~/.ssh/id_rsa
    keep_releases: 10                # ten releases to roll back to
    shared:                          # symlinked into every release
      - .env
      - var/log/
      - public/fileadmin/
      - public/uploads/

commands:
  - name: Clear TYPO3 cache
    run: ./{{config.bin-dir|vendor/bin}}/typo3 cache:flush

  - name: Database migrations
    run: ./{{config.bin-dir|vendor/bin}}/typo3 upgrade:run
```

[Full configuration guide →](/guide/configuration)

## What it looks like on the server

The familiar Deployer/Capistrano layout — point your web server at `current/` and never touch it
again.

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

## Eight steps, and the site only moves at step seven

1. **Scan files** — walks the source directory, respecting `.gitignore` and your exclude patterns
2. **Connect to server** — establishes the SSH connection
3. **Create release** — a new timestamped directory, e.g. `releases/20260109203841`
4. **Sync files** — transfers everything into that release
5. **Create symlinks** — links the shared files and directories
6. **Execute commands** — cache flush, migrations, warmup — inside the new release
7. **Activate release** — atomically repoints `current`; the site goes live
8. **Cleanup** — removes old releases, keeps the last N

A failure in steps 1–6 never reaches production: the broken release is simply never activated.

[Read the full sequence →](/guide/deployment-process)

## Ships from your pipeline too

::: code-group

```yaml [GitHub Actions]
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ochorocho/shippy-action@v0.0.1
        with:
          args: deploy production
```

```yaml [GitLab CI]
# .gitlab-ci.yml
deploy:
  image: ochorocho/shippy:latest
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  script:
    - shippy deploy production
```

:::

[CI/CD guide →](/guide/ci-cd)

## Also in the box

- **`shippy backup`** — a ZIP with a database dump and your shared files, with TYPO3 credentials
  auto-detected and cache tables excluded
- **`shippy gitlab:upload`** — push that archive to the GitLab package registry straight from a
  scheduled pipeline
- **`shippy rollback -l`** — list every release with its date, git commit and tag before you choose

[Command reference →](/reference/commands)
