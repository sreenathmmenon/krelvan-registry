# Krelvan Registry

The official capability marketplace for [Krelvan](https://github.com/sreenathmmenon/krelvan) — *own, run, and trust your own AI agents.*

This repo **is** the marketplace. There is no hosted marketplace server: a Krelvan
instance reads [`index.json`](./index.json) straight from GitHub raw, and anyone
publishes a capability by opening a pull request. Think *WordPress.org for agents*.

```
Krelvan "Discover" tab
        │  fetch
        ▼
https://raw.githubusercontent.com/sreenathmmenon/krelvan-registry/main/index.json
        │  install-by-definition (no code download — a signed manifest reference)
        ▼
the tool becomes a capability your agents can use, under the same
approval gates and signed audit trail as everything else.
```

---

## What's in here

| File | Purpose |
|---|---|
| [`index.json`](./index.json) | The registry index — every published capability. This is what Krelvan fetches. |
| [`capabilities/`](./capabilities) | The standalone YAML for each capability (the source of truth for the `yaml` field in the index). |

Each entry is **real and working** — a YAML wrapper over a real API, a real MCP
server, or a real provider deploy hook. No fabricated entries.

### Categories

- **Connectors** — MCP servers (GitHub, Filesystem, …). Every tool the server exposes becomes a capability.
- **Research** — read-only data fetchers (Hacker News, Weather, Wikipedia, web search).
- **Messaging** — notify a human/channel (Discord, …).
- **Deploy** — ship a site or app to a provider (Vercel, Netlify, Cloudflare Pages, Render, Railway).

---

## Capability schema

Each item in `index.json`'s `capabilities` array:

```jsonc
{
  "name": "deploy.vercel",                 // unique id
  "title": "Deploy to Vercel",             // display name
  "oneLiner": "Ship a production deploy…", // one-sentence description
  "category": "Deploy",                    // Connectors | Research | Messaging | Deploy | …
  "sideEffect": "write-irreversible",      // read | write-reversible | write-irreversible | spend | message-human | identity-mutation
  "tier": "official",                      // official | community
  "author": "Krelvan",
  "kind": "yaml",                          // "yaml" (HTTP wrapper) or "mcp" (server)
  "secretRefs": ["vercel-deploy-hook"],    // named secrets the user supplies (never inlined)
  "sourceUrl": "https://github.com/sreenathmmenon/krelvan-registry",
  "yaml": "name: deploy.vercel\n…"         // for kind:"yaml" — the full capability definition
  // or:  "mcp": { "name": "github", "command": "npx", "args": [...] }  // for kind:"mcp"
  // paid entries add:  "price": "from $5/mo", "licenseUrl": "https://…"
}
```

### YAML capability format

```yaml
name: deploy.vercel
description: Trigger a production deployment on Vercel via a Deploy Hook URL.
sideEffect: write-irreversible      # gates on the agent's autonomy
estimateCents: 0
http:
  url: "{{secret:vercel-deploy-hook}}"   # {{secret:name}} — resolved by the broker, never inlined
  method: POST
  headers:
    Content-Type: "application/json"
  body:
    ref: "{{input.ref}}"                 # {{input.field}} — from run state
input:
  ref:
    type: string
    description: Optional git ref to deploy.
successCodes: [200, 201, 202]
```

Only whitelist interpolation (`{{secret:…}}`, `{{input.…}}`) — **no `eval`, ever.**
Secrets are referenced by name and resolved by Krelvan's secret broker at call
time; the registry never contains a real key.

---

## Deploy capabilities — how to use one

The deploy capabilities use each provider's **deploy/build hook** (a unique URL you
create once that triggers a deployment when POSTed). Steps:

1. Install the capability from the Krelvan **Discover** tab (e.g. *Deploy to Vercel*).
2. In your provider, create the hook:
   - **Vercel** — Project → Settings → Git → Deploy Hooks
   - **Netlify** — Site → Build & deploy → Build hooks
   - **Cloudflare Pages** — Project → Settings → Builds & deployments → Deploy hooks
   - **Render** — Service → Settings → Deploy Hook
   - **Railway** — Account/Project → Tokens (uses the GraphQL API + service id)
3. Store it in Krelvan as the named secret (e.g. `vercel-deploy-hook`).
4. Give an agent the capability. Because deploys are `write-irreversible`, the
   agent will **pause for your approval** before shipping unless you've granted it
   full autonomy.

---

## Publish a capability

1. Fork this repo.
2. Add your capability YAML under `capabilities/<name>.yaml`.
3. Add a matching entry to `index.json` (`tier: "community"` for third-party).
4. Open a PR. Keep it **real and working** — reviewers install it to verify.

Rules:
- No secrets in the repo — reference them as `{{secret:name}}`.
- `sideEffect` must be accurate (it drives Krelvan's approval gating).
- One capability per entry; `name` must be unique.

---

*Part of [Krelvan](https://github.com/sreenathmmenon/krelvan). Open-source · self-hosted.*
