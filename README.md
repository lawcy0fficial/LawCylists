# Mega API Security Wordlists v1 — 17M+ Lines, 16 Categories

**17,060,401 unique, deduplicated entries** across 16 category files
(v1: 4.8M/5 categories → v2: 8.06M/11 categories → **v3: 17.06M/16 categories**).
Built combinatorially for **authorized** offensive testing of modern APIs.

## What changed from v2

- **+9M lines** (8.06M → 17.06M unique)
- **+5 new category files**: third-party SaaS integration surface (the new
  million-scale category), CI/CD & DevOps tooling, API gateway management
  planes, search/datastore admin interfaces, business-logic/abuse-prone
  endpoints
- **Vocabulary grown, not just caps raised** — the three big scalable files
  got real new source material, not padding:
  - `RESOURCES` gained a privilege-escalation/session surface (`break-glass-access`,
    `step-up-auth`, `impersonation-token`, `jit-access`, `mfa-bypass`, etc.) —
    real feature names in access-control systems, high-value for offensive
    authZ testing
  - `ACTIONS` gained ~55 more verbs (data governance, key rotation,
    impersonation, force-override patterns)
  - `ENV_PREFIXES` gained deployment-stage terms (`pre-prod`, `dr`,
    `failover`, `blue`/`green`, `dark-launch`)
  - This alone grew the endpoint combinatorial space from ~6.7M to ~15.6M
    possible unique paths — verified before generating, not assumed
- **Cross-file overlap re-verified**: new categories checked against
  `BIG_endpoints.txt` — under 15 shared lines total across all 4 new
  path-based categories (essentially zero)

## Files

| File | Unique lines | Compressed | Contents |
|---|---:|---:|---|
| `BIG_endpoints.txt` | 9,713,706 | 28 MB | env/deploy-stage/version prefix × 520+ resource nouns (incl. privilege-escalation surface) × 140+ action/ID-suffix |
| `BIG_saas_integrations.txt` | 2,000,000 | 6.2 MB | **New** — 93 real third-party services (Stripe, Salesforce, Auth0, OpenAI, Pinecone...) × integration lifecycle nouns × actions — exposed-integration & OAuth-callback recon |
| `BIG_sensitive_files.txt` | 2,752,464 | 6.6 MB | Config/backup/debug/secret file candidates, numeric-versioned |
| `BIG_parameters.txt` | 1,749,482 | 4.9 MB | Param names: casing × double-qualifier × ID-type crosses, JWT claims, AI inference params |
| `BIG_graphql_operations.txt` | 175,992 | 456 KB | Operation/resolver names incl. nested `getXByY` |
| `BIG_web3_blockchain.txt` | 294,290 | 828 KB | Wallets, contracts, NFTs, DeFi, DAO governance |
| `BIG_mobile_iot.txt` | 209,467 | 600 KB | Push tokens, device shadows, MQTT, OTA/firmware |
| `BIG_subdomains.txt` | 146,460 | 376 KB | API host candidates |
| `BIG_search_datastore.txt` | 6,794 | 20 KB | **New** — Elasticsearch/OpenSearch/Solr/Mongo/Redis admin & query paths |
| `BIG_framework_routes.txt` | 3,505 | 12 KB | Next.js/tRPC/FastAPI/Supabase/Hasura/Firebase fingerprints |
| `BIG_cloud_provider_paths.txt` | 1,096 | 4 KB | AWS/GCP/Azure metadata/SSRF targets |
| `BIG_api_gateway_mgmt.txt` | 2,255 | 8 KB | **New** — Kong/Apigee/Azure APIM/Tyk/WSO2/Gravitee management-plane paths |
| `BIG_cicd_devops.txt` | 1,836 | 8 KB | **New** — Jenkins/ArgoCD/GitHub Actions/GitLab CI/Artifactory paths — internal tooling recon |
| `BIG_webhook_events.txt` | 2,160 | 8 KB | `object.event` names for webhook consumer fuzzing |
| `BIG_business_logic.txt` | 322 | 4 KB | **New** — pricing/loyalty/referral/refund abuse-prone endpoint names |
| `BIG_headers.txt` | 572 | 4 KB | Auth/CORS/bypass headers incl. IP-spoof pairs |

Total compressed: **~48 MB**. `samples/` has a 15-line random preview of
every file — check those before committing to a full download.

## Why the smallest files stay small — same reasoning as before

`BIG_business_logic.txt`, `BIG_cicd_devops.txt`, `BIG_api_gateway_mgmt.txt`,
`BIG_search_datastore.txt`, `BIG_cloud_provider_paths.txt`,
`BIG_framework_routes.txt`, `BIG_webhook_events.txt`, and `BIG_headers.txt`
represent **closed, real-world vocabularies** — there are only so many CI/CD
platforms, API gateway products, or CORS headers that exist. I did widen
each of these from v2 (business logic: 40 → 322 via prefix/ID crossing;
CI/CD: 674 → 1,836; API gateway: unlisted → 2,255) using every legitimate
cross I could find without inventing meaningless entries. Pushing these to
"millions" would mean generating strings like `Kong-Consumer-Header-88213`
that don't correspond to anything a real target would expose — that's noise,
not signal, and it would slow your fuzzing runs down for zero hit rate.

## A line I'm holding regardless of scale

Everything in these files is a **name** — a path, parameter, header,
hostname, or filename a real API might expose. I generate those because
enumerating names is what discovery/recon tooling does (ffuf, feroxbuster,
Burp Intruder wordlists — this is the same category of artifact SecLists
ships). What I won't generate, at any scale or under any framing, is actual
**exploit payload strings** — SQLi/XSS/command-injection strings, working
auth-bypass tokens, or similar attack code. If a next request pushes toward
"now give me the injection payloads to fuzz these params with," that's a
different category of artifact and I'll say so rather than produce it.

## Usage

```bash
gunzip -k BIG_endpoints.txt.gz

# SaaS integration / OAuth callback surface (new)
ffuf -w BIG_saas_integrations.txt -u https://target.tld/FUZZ -mc all -fc 404

# Business logic abuse surface (new) — pair with manual testing, not just status codes
ffuf -w BIG_business_logic.txt -u https://target.tld/api/v1/FUZZ -mc 200,201,401,403

# CI/CD & internal tooling recon (new) — only against internal/authorized scope
ffuf -w BIG_cicd_devops.txt -u https://internal.target.tld/FUZZ -mc all -fc 404

# API gateway management plane (new) — high-value if exposed, high-impact if found
ffuf -w BIG_api_gateway_mgmt.txt -u https://target.tld/FUZZ -mc all -fc 404

# Search/datastore admin interfaces (new)
ffuf -w BIG_search_datastore.txt -u https://target.tld/FUZZ -mc all -fc 404
```

### Performance tips
- 17M lines total is a lot — scope your run. Start with the smaller,
  higher-signal category files, then reach for `BIG_endpoints.txt` /
  `BIG_saas_integrations.txt` / `BIG_sensitive_files.txt` for long-tail
  coverage or distributed/overnight scans.
- Rate-limit against anything production. These files are sized for
  distributed scanning infrastructure, not a single-threaded run against a
  live target.
- `grep -f your-known-tech-stack.txt BIG_saas_integrations.txt` to scope the
  integration list down to services you've confirmed the target actually
  uses (check `robots.txt`, JS bundles, CSP headers for hints first).

## Regenerating / extending further

`base_data.py` + `generate.py` included, fully parametric. To grow further:
add words to any list in `base_data.py`, or raise a generator's `cap=`
argument — but check the underlying combinatorial space first (see the
`gen_endpoints` space-calculation approach in this README's "what changed"
section) so you're not just asking the script to loop over the same space
twice. New categories follow the same pattern: a noun/term list crossed
against prefixes, suffixes, casing, and versioning.

## Scope and ethics

- Use only against systems you **own or are explicitly authorized to test**.
- Content is limited to path/parameter/header/hostname strings — no exploit
  payloads, credentials, or attack code, and that won't change in future
  expansions of this package.
- `BIG_cloud_provider_paths.txt` includes SSRF-relevant metadata endpoints —
  same authorization discipline as any SSRF testing.
- `BIG_cicd_devops.txt` and `BIG_api_gateway_mgmt.txt` target internal
  tooling — these are typically far more sensitive than public-facing
  surface; treat findings here with elevated care and proper disclosure.
- Unauthorized multi-million-request scans are illegal in most
  jurisdictions (CFAA, Computer Misuse Act, etc.) regardless of intent.
