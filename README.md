# zharconsulting-site

Digital garden for Zhar Consulting, built with [Quartz v5](https://quartz.jzhao.xyz) from an Obsidian vault.

Content lives in `content/` (currently the `Contents/` folder of the vault). `content/index.md` is the landing page.

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
```

## Deploying

Pushing to `main` triggers `.github/workflows/deploy.yml`, which builds the site to `public/` and deploys it to GitHub Pages at <https://zesky665.github.io/zharconsulting-site/>.

In the repo settings on GitHub: Settings -> Pages -> Source must be set to **GitHub Actions**.

Site configuration (title, theme, plugins, baseUrl) is in `quartz.config.yaml`. See `docs/` for the full Quartz documentation.
