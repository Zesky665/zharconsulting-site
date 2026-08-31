# zharconsulting-site

Digital garden for Zhar Consulting, built with [Quartz v5](https://quartz.jzhao.xyz) from an Obsidian vault.

Content lives in `content/` — mirrored from the `Contents/` and `Site/` folders of the vault. `content/index.md` is the landing page.

## Local development

Requires **Node >= 22** (e.g. `nvm use 22`):

```bash
npm install
npx quartz build --serve   # serves at http://localhost:8080
```

## Updating content

Copy or edit Markdown under `content/` and rebuild. To sync from the Obsidian vault:

```bash
rsync -a --delete --exclude='.DS_Store' ~/Documents/obsidian-vault/Contents/ content/Contents/
rsync -a --delete --exclude='.DS_Store' ~/Documents/obsidian-vault/Site/ content/Site/
```

Known issue: macOS ships an ancient rsync (2.6.9) whose dry-run (`-n`) can wrongly report no changes — verify syncs with `diff -rq` if unsure.

## Deploying

Pushing to `main` triggers `.github/workflows/deploy.yml`, which builds the site to `public/` and deploys it to GitHub Pages.

The site is served at <https://zharconsulting.com> (custom domain, CNAME emitted by the cname plugin; `baseUrl` in `quartz.config.yaml` must match).

Required DNS records at the domain provider:

| Record  | Name | Value                    |
| ------- | ---- | ------------------------ |
| A       | `@`  | `185.199.108.153`        |
| A       | `@`  | `185.199.109.153`        |
| A       | `@`  | `185.199.110.153`        |
| A       | `@`  | `185.199.111.153`        |
| CNAME   | `www` | `zesky665.github.io`    |

Once DNS resolves, enable "Enforce HTTPS" under Settings -> Pages.

Site configuration (title, theme, plugins, baseUrl) is in `quartz.config.yaml`. See `docs/` for the full Quartz documentation.
