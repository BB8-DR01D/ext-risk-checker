## Repository layout

```
your-repo/
├── index.html          ← The entire app (single file, no build step)
├── extensions-db.json  ← Your database (grows automatically on every lookup)
└── README.md

cloudflare-worker/      ← Deploy separately to Cloudflare
├── worker.js           ← The secure proxy (holds your GitHub token)
├── wrangler.toml       ← Worker config (non-secret values only)
└── SETUP.md            ← Step-by-step deployment guide
```

---

## How it works

```
User enters ID
      │
      ▼
Check extensions-db.json (via Worker)
      │
      ├── Found → score it → show result
      │
      └── Not found → fetch Chrome Web Store (via Worker)
                           │
                           ▼
                     Parse + score → save to DB → show result
```

The **risk score (0–100)** is computed entirely in the browser from five factors:

| Factor | Weight | What it measures |
|--------|--------|-----------------|
| Permissions | 35% | High-risk permissions (debugger, proxy, `<all_urls>`, etc.) |
| Developer trust | 25% | Verified / known publishers vs anonymous |
| Extension age | 15% | Brand-new extensions are higher risk |
| Install base | 10% | More users = more community scrutiny |
| Maintenance | 10% | Abandoned extensions may have unpatched issues |

Manual trust overrides (trusted / caution / malicious) adjust the final score and display a prominent notice.

---

## Database (`extensions-db.json`)

The database is pre-seeded with:

- **12 trusted** extensions — Google, Microsoft, Bitwarden, LastPass
- **2 caution** — Honey (PayPal), Hotspot Shield
- **14 malicious** — DataSpii spyware + 13 entries from your June 2026 blocklist import (Bundling Unwanted Software, Policy Violations, Malware, Search Hijacking, WhatsCluster PUS cluster)

Every new lookup automatically adds an entry. You can enrich entries via the in-app admin panel.

### Entry structure

```json
{
  "id": "aapbdbdomjkkjkaonfhkkikfgjllcleb",
  "name": "Google Translate",
  "developer": "Google LLC",
  "developerVerified": true,
  "publishedDate": "2010-02-01",
  "lastUpdated": "2024-09-01",
  "users": 20000000,
  "permissions": ["activeTab", "storage", "contextMenus"],
  "category": "Productivity",
  "trustLevel": "trusted",
  "trustReason": "Official Google extension",
  "storeUrl": "https://chrome.google.com/webstore/detail/aapbdbdomjkkjkaonfhkkikfgjllcleb",
  "source": "seed",
  "blocklist_reason": null,
  "blocklist_source": null,
  "insert_date": null
}
```

| Field | Values | Notes |
|-------|--------|-------|
| `trustLevel` | `trusted` / `caution` / `malicious` / `unknown` | Drives the risk override |
| `source` | `seed` / `store` / `blocklist` / `manual` | How the entry got in |
| `blocklist_source` | URL or text | Only rendered as a link if it's `https://` |
