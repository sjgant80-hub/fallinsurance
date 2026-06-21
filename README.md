# FallInsurance

**Sovereign single-file UK GI insurance broker policy management anchor.** One HTML file, runs entirely in the browser. Multi-firm, multi-adviser, multi-client, multi-policy. IDD demands-and-needs, IPID tracking, CASS 5 client money configuration, commission accounting, renewal pipeline, claims register, audit chain.

`version 1.0.0` · `prime 829` · bundle anchor for `fall-insurance` (with `fallinsuranceonboard` 839, `fallinsurancepaper` 853, `fallinsurancepractice` 857)

---

## For brokers (the people who use this)

You run a small UK general-insurance brokerage. Maybe you're 1-10 advisers. You're FCA-regulated (DA or AR through a network like Cobra / Bspoke / Movo / Bravo). You've got a spreadsheet of policies, a folder of IPIDs, a calendar full of renewals, and a vague feeling the regulator will eventually ask for your demands-and-needs evidence.

**FallInsurance is one file that holds the whole thing together.**

Open `index.html` in Chrome. First launch walks you through firm setup (FCA ref, PI cover, CASS model, IDD compliance declaration) and your first adviser (SM&CR role, CPD hours). A demo policy seeds so you can poke around — it auto-deletes when you add your first real one.

- **Policy list** sidebar — search/filter by client, ref, insurer, product class (commercial-property / EL / PL / motor-fleet / cyber / PI / D&O / landlord / home / travel / health / pet), status, renewal window.
- **Critical-date flags** on renewals: 60-day amber, 30-day red, 14-day critical, 7-day black.
- **Policy detail tabs** — Overview, Cover, Demands & Needs (IDD Art 20 questionnaire that compiles a delivered statement), IPID (delivery + hash tracking), Commission (gross/IPT/fee/commission% calc), Mid-Term Adjustments, Claims register, Timeline.
- **Bind discipline** — you can't move a policy to `in-force` without a recorded IPID delivery and a D&N statement >30 chars. Audit chain logs every change.
- **Renewal pipeline** dashboard — four buckets (7/14/30/60 day), CSV export, broadcast renewal.due to the bundle.
- **Compliance snapshot** — PI expiry, MIPRU 3.2.7R minimums, CPD per adviser, CASS model.
- **T0** — 14 offline rules on IDD Article 20, IPID, CASS 5, PI cover, FOS limits, AR vs DA, Consumer Duty, premium financing, cyber market, schemes admin, MTAs, customer category, ICOBS, IDD overview.
- **T3** — bring-your-own Anthropic API key for case-specific consultation; sends active policy context with each turn.
- **Audit chain** — every mutation hashed (SHA-256), prev-linked, capped at 100k, exportable as JSON. FCA SYSC 6-year retention.
- **Bundle mesh** — `BroadcastChannel('fall-insurance')` syncs with onboard/paper/practice tools when they're open.

Your data never leaves the machine. No server. No analytics. No CDN dependencies (Anthropic API is opt-in for T3 only).

### Daily use

1. **Morning** — open the Renewals tab. Work the black bucket (≤7d) and the critical bucket (≤14d). One click each opens the policy.
2. **New enquiry** — `+ Policy` (top-right) → fill defaults → land in the Overview tab. Walk the D&N questionnaire while on the call. Compile the statement. Issue IPID, record the delivery + hash. Then move status to `in-force`.
3. **MTA mid-year** — open the policy → MTAs tab → log the change with the premium delta. Audit chain captures the change reasoning.
4. **Claim notification** — Claims tab → notify with reserve. Broadcast goes to `fallinsurancepaper` for FNOL letter; `fallinsurancepractice` picks up the financial side.
5. **End of month** — Audit tab → Export JSON. Drop it in your compliance folder.

### Disclaimer

FallInsurance is a tool for UK FCA-regulated insurance brokers. It assists with policy management, IDD-compliant demands-and-needs documentation, CASS 5 client money tracking, and commission accounting. **It is not a quote/bind/issue platform** — insurer integrations remain the broker's responsibility. **Sovereign — client data never leaves the device.**

---

## For builders (the people who fork or extend this)

### Architecture

Single HTML file, ~one CSS block, one `<script>`, no build step. Browser-native:

- **IndexedDB** primary storage. Stores: `firms`, `advisers`, `clients`, `policies`, `clientAccount`, `audit`, `settings`. KeyPath `id` across all.
- **`BroadcastChannel('fall-insurance')`** for bundle mesh. Event types: `policy.created`, `policy.updated`, `policy.lapsed`, `policy.claimed`, `renewal.due` (with `window: 60|30|14|7`), `commission.received`, `client.*`, `adviser.*`, `firm.*`, `sync.request`, `sync.snapshot`. Boot emits `sync.request`; any open peer replies with `sync.snapshot`. 300ms debounced on rapid writes.
- **`BroadcastChannel('fall-signal')`** for cross-bundle estate signal (boot ping, prime 829).
- **`window.KONOMI`** shim: `{tool, version, prime:829, tier:'sovereign', bundle:'fall-insurance', bundleMembers:[…], meshes:[…]}`.
- **Audit chain** — Mansoor P3 extended. Each entry `{i, ts, tool, adviserId, clientId, policyId, action, reasoning, configVersion, prevHash, docHash, payload}`. SHA-256 via WebCrypto. Cap 100k FIFO.
- **PWA manifest** via inline `data:` URL.
- **Aesthetic** — oxblood `#8b1a1a` / brass `#b8974a` / cream `#e6e1d6` / void `#0b0a0f`. Libre Baskerville serif, DM Sans body, IBM Plex Mono accents.

### Schema conformance

Conforms to `INSURANCE-BUNDLE-SHARED-SCHEMA.md` v1.0 (Policy / InsuranceClient / InsuranceAdviser) and extends the base IFA `Client/Firm/Adviser` shape from `IFA-BUNDLE-SHARED-SCHEMA.md`. Other bundle members (`fallinsuranceonboard`, `fallinsurancepaper`, `fallinsurancepractice`) read/write the same records over the mesh.

### Files

```
index.html   — the deliverable, single file
README.md    — this file
LICENSE      — MIT
.nojekyll    — disables Jekyll on GitHub Pages
```

### Boot sequence

```
openDB()
  → firms.length === 0 ? renderOnboard()
  : load all stores → set STATE → boot mesh sync.request
  → signal('boot') → audit('boot') → render()
```

### Extension points

- **`PRODUCT_CLASSES`** + `PRODUCT_LABEL` — add a class, no other changes needed.
- **`T0_RULES`** array — add an offline rule (object: `{q, a}`).
- **`STORES`** — add an IDB store (and bump `DB_VERSION`).
- **`meshSend()`** / `meshOnMessage()` — add a mesh event type.
- **Forkable brand** — rename in source (`TOOLNAME`, `<title>`, brand mark). Forks remain MIT.

### Sovereignty rules

- One HTML file
- IDB primary, localStorage fallback only as last resort
- No npm, no CDN (except optional Anthropic API for T3)
- No analytics, no telemetry
- 14-pt gate per estate doctrine
- Two-audience README (this file)

### Verify

```
node -e "const fs=require('fs');const h=fs.readFileSync('index.html','utf8');new Function(h.match(/<script>\s*'use strict';([\s\S]*?)<\/script>/)[1]);console.log('OK')"
```

Open in Chrome 113+. First launch should show onboarding. After completing onboarding you should see a demo policy in the sidebar (Patel Wealth Ltd commercial property) and a populated dashboard.

### Deploy

```
git init && git add . && git commit -m "fallinsurance v1.0.0"
git remote add origin git@github.com:sjgant80-hub/fallinsurance.git
git push -u origin main
gh repo edit sjgant80-hub/fallinsurance --enable-pages --pages-branch=main --pages-build-type=legacy
```

The `.nojekyll` file is what makes GitHub Pages stop trying to be clever with `_` directories.

---

`◊ 829 · sovereign`
