# shippy-website

Marketing site and documentation for [Shippy](https://github.com/ochorocho/shippy), the deployment
tool for Composer-based PHP projects.

Built with [VitePress](https://vitepress.dev/). All content is Markdown under `docs/`.

## Develop

```bash
npm install
npm run docs:dev      # http://localhost:5173/shippy-website/
```

## Build

```bash
npm run docs:build    # output: docs/.vitepress/dist
npm run docs:preview  # serve the build under the /shippy-website/ base
```

The build fails on dead internal links, so a green build is also a link check.

## Structure

```
docs/
├── index.md                     # landing page (layout: home)
├── guide/                       # narrative documentation
│   ├── index.md                 # what is Shippy?
│   ├── installation.md
│   ├── quick-start.md
│   ├── configuration.md
│   ├── ssh.md
│   ├── deployment-process.md
│   ├── ci-cd.md
│   ├── examples.md
│   └── contributing.md
├── reference/                   # lookup tables
│   ├── commands.md
│   └── configuration.md
├── public/logo.svg              # from ochorocho/shippy, images/logo.svg
└── .vitepress/
    ├── config.ts                # nav, sidebar, SEO, search
    └── theme/                   # brand colours from the logo palette
```

The documentation content mirrors the
[README of ochorocho/shippy](https://github.com/ochorocho/shippy/blob/main/README.md), split into
pages. When that README changes, update the corresponding page here.

## Deployment

Pushing to `main` runs `.github/workflows/deploy.yml`, which builds the site and publishes it to
GitHub Pages at <https://ochorocho.github.io/shippy-website/>.

Enable it once under **Settings → Pages → Build and deployment → Source: GitHub Actions**.

Serving from a custom domain later means setting `base: '/'` in `docs/.vitepress/config.ts`, updating
`sitemap.hostname`, and adding `docs/public/CNAME`.

## License

MIT, matching the upstream project.
