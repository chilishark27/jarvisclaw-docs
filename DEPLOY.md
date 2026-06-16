# Docs Site Deployment

## Cloudflare Pages Setup

1. In Cloudflare Dashboard → Pages → Create a project
2. Connect your Git repository
3. Build settings:
   - **Build command:** `cd docs-site && npm install && npm run build`
   - **Build output directory:** `docs-site/.vitepress/dist`
   - **Root directory:** `/` (project root)
4. Custom domain: `docs.jarvisclaw.ai`

## DNS Setup

Add a CNAME record in Cloudflare DNS:

```
docs.jarvisclaw.ai → your-project.pages.dev
```

## Local Development

```bash
cd docs-site
npm install
npm run dev
# → http://localhost:5173
```

## Manual Deploy (Alternative)

If not using Pages CI:

```bash
cd docs-site
npm run build
# Upload .vitepress/dist/ to Cloudflare R2 or any static host
```

## Structure

```
docs-site/
├── .vitepress/config.ts  — sidebar, nav, theme config
├── index.md              — homepage
├── quickstart.md         — getting started
├── mcp.md                — MCP configuration
├── agent-registry.md     — agent registry API
├── discovery.md          — x402 discovery protocol
├── x402.md               — agent payments
├── sdk.md                — Python & Go SDK
├── api/                  — API reference pages
│   ├── chat-completions.md
│   ├── image-generation.md
│   └── ...
└── package.json
```
