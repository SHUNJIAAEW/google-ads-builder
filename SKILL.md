---
name: google-ads-builder
description: >
  Point Claude at a website URL and it builds a ready-to-launch Google Search Ads
  campaign — keyword themes grouped into tight ad groups, a full negative-keyword
  list, Responsive Search Ads (15 headlines + 4 descriptions each) written to Google's
  character limits and policy, plus sitelinks/callouts and recommended match types,
  bidding, and budget split. Exports a Google Ads Editor CSV you can import in one click,
  and renders a campaign dashboard. Trigger on "build my Google ads", "build google ads",
  "build my Google search campaign", "build search ads", "google ads from my website",
  "make me a search campaign", "generate google search ads".
  日本語対応: サイトのURLからGoogle検索広告（リスティング広告）のキャンペーン一式
  ——広告グループ設計・キーワードとマッチタイプ・除外キーワードリスト・レスポンシブ検索広告
  （見出し15本＋説明文4本、全角文字数の上限を厳守）・サイトリンクとコールアウト・入札戦略と
  予算配分——を作成し、Google Ads Editor にそのままインポートできるCSVとダッシュボードを出力する。
  次のように言われたら発火: 「Google広告を作って」「Google広告を作成して」「検索広告を作って」
  「リスティング広告を作って」「リスティング広告を組んで」「広告キャンペーンを作って」
  「このサイトで広告を出したい」「サイトのURLから広告を作って」「検索広告のキャンペーンを設計して」
  「Google広告のCSVを作って」「広告文を作って」。
allowed-tools: Bash(python3 *) Read Write WebFetch WebSearch
---

# Google Ads Builder

**The problem this solves:** spinning up a Google Search campaign from scratch is hours of tedium — keyword research, grouping into tight ad groups, writing 15 headlines and 4 descriptions per ad group inside Google's exact character limits, building a negative list so you don't burn budget on junk clicks, and wiring up extensions. Most people either pay an agency for the setup or let Google's "Smart" defaults build a sloppy campaign that leaks spend.

This skill does the whole build from **one input: your website URL.** It reads the site, works out what you sell and who for, and produces a complete, launch-ready Search campaign — structured the way a good paid-search manager would build it — exported as a **Google Ads Editor CSV you import in one click**, plus a visual dashboard.

**What it is NOT:** it does not spend money, connect to your Google Ads account, or need any API key. It builds the campaign *draft*; you review it, add conversion tracking + real budgets, and launch. Search-volume figures are directional estimates (no paid keyword-tool API) — validate the final keyword list in Google Keyword Planner before scaling spend.

## 0 — Intake

Get the **website URL** (required). Then ask (don't skip — they change the whole build):

1. **What's the primary conversion?** purchase / lead form / phone call / booking. (Shapes the CTAs and bidding.)
2. **Target geography?** country / region / city radius. (Default: the site's apparent market.)
3. **Rough monthly budget?** (Shapes how many ad groups + match-type aggressiveness. Optional.)
4. **Any competitor brands to target, or brand terms to protect?** (Enables competitor + brand ad groups. Optional.)

Save intake to `./search-ads-run/brief.json`.

## 1 — Read the site

Fetch the homepage plus the key pages (products/services, pricing, about) with your web tools. Extract and note in `brief.json`:

- **What they sell** (products/services, categories)
- **Core value props + differentiators** (free shipping, guarantee, made in X, awards, speed)
- **Price points / offers** (so ad copy can lead with a real hook)
- **Proof** (reviews, customer count, notable clients)
- **Geographies + audience** signals

If the site is JS-heavy and a raw fetch returns an empty shell, use a headless/browser web tool. If you can't read the site at all, stop and say so — a campaign built from a guess isn't worth shipping.

## 2 — Design the campaign structure

Reason about the account **before** writing copy. Build tightly-themed ad groups — never one giant bucket. Standard shape:

- **Category / high-intent** — bottom-funnel buyers (`buy <product>`, `<product> for <use>`, `best <category>`). One ad group per distinct product line or theme.
- **Problem-aware** — searchers describing the pain your product solves (`how to <problem>`, `<problem> solution`).
- **Brand** — your own brand terms (cheap, high-converting; defends against competitors bidding on you).
- **Competitor** *(optional, only if provided/derivable)* — competitor brand terms. Flag the higher CPC + policy limits (no using competitor brand in ad text).

Each ad group: 5–15 tightly related keywords, mostly **phrase** and **exact** match (broad only with Smart Bidding + tight negatives). Note match type per keyword.

**Landing page per ad group.** If the site has dedicated pages per product/service/symptom, set `final_url` on the ad group to that page instead of dumping every click on the homepage — it lifts both Quality Score and conversion rate. Only point at pages you have actually confirmed return 200.

## 3 — Negative keywords

Build a negative list so budget isn't wasted:

- **Universal junk:** `free`, `jobs`, `salary`, `diy`, `how to make`, `used`, `torrent`, `pdf` (tune to the business).
- **Business-specific:** irrelevant meanings, wrong audience, out-of-scope products.
- **Match-level:** which negatives are exact vs phrase.

## 4 — Write the Responsive Search Ads

One RSA per ad group. Follow Google's spec **exactly**:

- **15 headlines**, each **≤ 30 characters**. Mix: keyword-matching, benefit, offer/price, CTA, proof, differentiator. Pin the brand or primary keyword to Position 1 only where it must always show.
- **4 descriptions**, each **≤ 90 characters**. Lead with the benefit/offer; end with a CTA.
- **Policy-safe:** no excessive capitalization, no unsupported superlatives ("#1", "best" without proof), no phone numbers in headlines, no repeated punctuation. Every claim must be supportable from the site.
- Include the ad group's core keyword in ≥ 3 headlines for Quality Score.

Character counts are non-negotiable — the export/render step flags any overflow, but write them correct the first time.

**Double-byte languages (Japanese / Chinese / Korean):** Google counts each full-width character as **2 units**, so the real limits are **15 full-width characters per headline** and **45 per description** (half of 30 / 90). Mixed half-width Latin/digits still count as 1 each. The renderer already measures width this way and flags overflow, but write Japanese headlines to the 全角15字 / 説明文全角45字 limit from the start. Also localize the negative-keyword junk list (`求人`, `採用`, `年収`, `無料`, `中古`, `使い方`, `自分で`, `evaluation`, etc.) to the business.

## 5 — Extensions + settings

- **Sitelinks** (4): real pages from the site with a 25-char text + two 35-char descriptions each.
- **Callouts** (4–6): 25-char benefit snippets (`Free Shipping`, `30-Day Returns`).
- **Structured snippets:** a header (e.g. Types, Brands) + values.
- **Settings recommendation:** match-type strategy, **bidding** (Maximize Conversions to start, tCPA once you have data), **budget split** across ad groups, geo, ad schedule, network (Search only — Display off).

## 6 — Write campaign.json

Write the full structured campaign to `./search-ads-run/campaign.json`:

```jsonc
{
  "business": "name",
  "url": "https://...",
  "goal": "purchase|lead|call|booking",
  "geo": "...",
  "budget_monthly": 3000,
  "settings": { "bidding": "...", "networks": "Search only", "schedule": "...", "notes": "..." },
  "ad_groups": [{
    "name": "Category — <theme>",
    "theme": "high-intent|problem-aware|brand|competitor",
    "final_url": "https://.../deep-page",   // optional — falls back to the campaign url
    "keywords": [{ "text": "buy blue widgets", "match": "phrase" }],
    "rsa": { "headlines": ["...15, each <=30..."], "descriptions": ["...4, each <=90..."], "paths": ["path1", "path2"] }
  }],
  "negatives": [{ "text": "free", "match": "phrase" }],
  "extensions": {
    "sitelinks": [{ "text": "...", "desc1": "...", "desc2": "...", "url": "..." }],
    "callouts": ["Free Shipping", "..."],
    "snippets": { "header": "Types", "values": ["...", "..."] }
  },
  "caveats": ["honest limits of this build"]
}
```

`caveats` is mandatory. Always include: *"This is a launch-ready draft, not a live campaign — add conversion tracking and set real budgets before launch. Search-volume and CPC are directional; validate in Keyword Planner."* Plus run-specific limits (site partially readable, no competitor data, etc.).

## 7 — Render the dashboard + export the CSV

```
python3 ${CLAUDE_SKILL_DIR}/scripts/render_report.py
```
(If `CLAUDE_SKILL_DIR` isn't set, use the folder containing this SKILL.md.)

Reads `campaign.json`, writes to `./search-ads-run/`:
- **`google-ads-editor.csv`** — import-ready (Campaign / Ad Group / Keyword / Match Type rows + RSA rows + negatives). Import via Google Ads Editor → Account → Import.
- **`campaign-dashboard.html`** — dark-mode visual: structure tree, per-ad-group keyword + RSA preview (with live character counts, red if over limit), the negative list, extensions, and the settings recommendation. Opens automatically (`--no-open` to skip).

The script only renders + exports; the campaign design is model judgment.

## 8 — Report in chat

Close with: the campaign at a glance (N ad groups, M keywords, the budget split); the single sharpest keyword theme (where the cheapest high-intent conversions are); the one thing to fix before launch (usually: install conversion tracking); and the one-sentence takeaway — *"a paid-search manager's first-week build, from just your homepage URL."* Tell them the CSV path and that it imports straight into Google Ads Editor.

---

## Notes
- **Zero API keys, no Google Ads login.** The site is public and read with your built-in web tools; the renderer/exporter is stdlib Python.
- **Tight ad groups beat big ones.** Quality Score, CTR, and CPC all reward keyword↔ad relevance. Never dump every keyword in one group.
- **The build is the product.** The script only renders + exports; the keyword strategy, grouping, and ad copy are model judgment — written to Google's real character limits and policy every time.
- **Tone of the output:** a sharp paid-search manager handing over a launch-ready account, not a keyword dump. Every ad group has a reason to exist; every headline earns its place.
