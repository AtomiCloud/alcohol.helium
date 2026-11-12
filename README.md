# alcohol.helium — t.lazytax.club

The **permanent NFC tag domain** for LazyTax. Physical stickers are written and
locked with `https://t.lazytax.club/t/{tagId}` — those URLs can never be
rewritten, so this domain must never change or break.

**This repo is frozen on purpose.** It is a single landscape-agnostic (arceus)
Cloudflare worker serving three static files and nothing else. There is no
application code, no build step, and no per-landscape environment: the
installed neon app flavor (pichu/pikachu/raichu) is the environment selector,
and each landscape's zinc resolves tag ids against its own database.

## What it serves

| Path | Purpose |
|---|---|
| `/.well-known/apple-app-site-association` | iOS Universal Links — lets the neon app claim `/t/*` taps and `/f/*` invite links. **This is the only home of this file.** |
| `/.well-known/assetlinks.json` | Android App Links — same, for the 3 flavor packages |
| `/t/*` | 302 → `https://lazytax.club/t/…` — browser fallback for phones without the app (argon's marketing pitch page). 302 not 301 so the target stays changeable. |
| `/f/*` | 302 → `https://lazytax.club/f/…` — friend invite links. With the app they open neon to accept the invite; without it, argon's invite page (with the link-preview tags). |

App-installed taps never hit this worker at tap time — iOS/Android route to the
app locally from the verification files fetched at install/update.

## Deploy

Push to `main` → GitHub Actions runs `wrangler deploy` (live immediately — no
staging; the repo ~never changes). Secrets: `CLOUDFLARE_API_TOKEN`,
`CLOUDFLARE_ACCOUNT_ID`. The custom domain `t.lazytax.club` is declared in
`wrangler.toml` (`custom_domain = true`) and attaches on deploy.

## Verify

```sh
curl -sI https://t.lazytax.club/.well-known/apple-app-site-association   # 200, application/json
curl -sI https://t.lazytax.club/t/abc12345                               # 302 → lazytax.club/t/abc12345
curl -s  https://app-site-association.cdn-apple.com/a/v1/t.lazytax.club  # Apple CDN view (may lag)
```

## Changing the app id lists

Adding/removing an app flavor means editing both well-known files here — and
remembering that iOS only refetches AASA on app install/update. Related config
lives in `alcohol.neon` (`Runner.entitlements`, `AndroidManifest.xml`,
`config/base.yaml` → `nfcTagBaseUrl`).

Plan/design: ClickUp [86eybdnh1](https://app.clickup.com/t/86eybdnh1).
