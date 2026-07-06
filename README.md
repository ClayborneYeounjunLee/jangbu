# Jangbu (장부) 📒

> A light, beautiful expense tracker web app that fits a day's income and spending onto a single calendar.
> It tints the calendar with each day's net amount, automatically expands **fixed recurring items** like salary, savings deposits, and subscriptions, and converts multiple currencies (₩/$/A$/₱) to KRW with live exchange rates for monthly, quarterly, and yearly analysis.

🔗 **Live:** https://clayborneyeounjunlee.github.io/jangbu/

A sibling app in the `moa` collection, it is a pure frontend app contained entirely in a single HTML file (just `index.html`). It runs in the browser alone — no build tools, bundlers, or backend server — and sign-in is optional (guest mode supported).

---

## ✨ Key Features

- **📅 Calendar-centric ledger** — Each date cell of the monthly calendar shows a summary of that day's **net amount (income − expenses)**. A positive net tints the cell green, a negative one red (intensity scaled to the month's largest absolute value), and large amounts are abbreviated like `1.2만` / `3.4억` (`K`/`M` in English).
- **Date detail sheet** — Tapping a date slides up a bottom sheet showing all of that day's items (regular transactions + recurring occurrences) along with income/expense/net totals, with add, edit, and delete available right there.
- **➕ Quick entry (FAB)** — A floating button at the bottom right of the calendar screen instantly adds a transaction for **today's date**.
- **🔁 Fixed recurring items** — Register regularly occurring income/expenses like salary, savings deposits, and subscriptions as rules. Supports **monthly (day of month) / weekly (day of week) / yearly (month and day)** cycles, with start and end dates to limit the effective period. Once registered, they are automatically expanded into the calendar and analysis without creating actual data (calendar cells show a 🔁 marker).
- **📊 Monthly/quarterly/yearly analysis** — Period summaries of income, expenses, and net; a net **trend bar chart** (monthly = per day, quarterly = per month, yearly = 12 months); and **category share bars** for expenses/income.
- **💱 Multi-currency + live exchange rates** — Enter each item in KRW, USD, AUD, or PHP. Foreign currencies are converted to KRW at live rates and reflected in totals and analysis, with an "≈ ₩…" preview shown even while typing.
- **☁️ Cloud storage or this-device-only** — Sign in with Google to store data in Firestore and continue across devices, or use **guest mode** without signing in to store data only on this device (localStorage).
- **🌗 Dark/light theme · 🌐 Korean/English** — Instant switching via toggles, automatic system-preference detection, and choices saved locally.
- **⬇ Backup export** — Download the entire state as a `jangbu-backup-YYYY-MM-DD.json` file.
- **📱 PWA feel** — Home-screen meta tags (`apple-mobile-web-app-*`), safe-area support, mobile-first layout (max width 520px).

---

## 🧱 Tech Stack / Languages

| Item | Details |
|---|---|
| Languages | **Pure HTML + CSS + JavaScript (ES modules)** — no frameworks |
| File layout | **Single file `index.html`** (about 1,195 lines; markup, styles, and scripts inline) |
| Build tools | **None** — no bundler, transpiler, or `package.json`. The browser runs the file as-is |
| Scripting | `<script type="module">` — the Firebase SDK is loaded on demand via **dynamic `import()`** |
| Styling | Inline `<style>` + **CSS custom properties (variables)** for the light/dark themes, using `color-mix()` and `env(safe-area-inset-*)` |
| Fonts | System font stack — `Pretendard`, `Apple SD Gothic Neo`, `Malgun Gothic`, `Noto Sans KR`, `sans-serif` (no web-font CDN; local fonts preferred) |
| Icons | Emoji + inline SVG favicon (data URI) |
| Backend | None (static). Data lives in Firestore or localStorage |

### CDN Libraries / SDKs

| Library | Version | How it loads | Purpose |
|---|---|---|---|
| Firebase App | 12.14.0 | `https://www.gstatic.com/firebasejs/12.14.0/firebase-app.js` (dynamic import) | App initialization |
| Firebase Auth | 12.14.0 | `firebase-auth.js` | Google sign-in (popup/redirect) |
| Firebase Firestore | 12.14.0 | `firebase-firestore.js` | Cloud storage + offline persistent cache |

> The Firebase version is managed by the code constant `FB_VER = "12.14.0"`, so all three modules load the same version.

---

## 🏗️ System Architecture

### Single file · Screen switching

It is an SPA that switches screens by **showing/hiding DOM elements (`.hidden`)** with no router. There are three top-level "screens".

```
screen-loading  (spinner)  →  screen-auth  (sign-in/guest)  →  #app  (main screen)
```

Inside `#app` are four tab screens (`.tabpane`) navigated via the bottom tab bar.

| Tab | Screen id | Render function | Description |
|---|---|---|---|
| 📅 Calendar | `screen-calendar` | `renderCalendar()` | Monthly calendar + bottom monthly totals, FAB shown |
| 🔁 Recurring | `screen-recurring` | `renderRecurring()` | This month's expected amount + fixed income/expense lists |
| 📊 Analysis | `screen-analysis` | `renderAnalysis()` | Month/quarter/year segments + trend + category bars |
| ⚙️ Settings | `screen-settings` | `renderProfile()` / `renderRates()` | Profile, theme, language, exchange rates, backup, account |

### Boot flow (IIFE at the bottom of the file)

1. `showLoading()` — show the loading screen.
2. `loadRatesCache()` — load the exchange-rate cache from localStorage; `applyLang(L)` — apply the language.
3. `fetchRates()` — try to fetch the latest rates in the background (re-render on success).
4. `initFirebase()` — dynamically import and initialize the three Firebase modules (the app continues even on failure).
5. On Firebase success, subscribe to `onAuthStateChanged`:
   - Signed-in user → `startCloud(user)` (load the document from Firestore)
   - Otherwise, if the previous mode was `guest`, `startGuest()`; else `showAuth()`
6. On Firebase failure (offline, etc.), fall back to the guest/auth screen.

### State Management

- All data lives in a single global `let state` (no external state library).
- Render functions read `state` and regenerate `innerHTML` each time — **imperative rendering**.
- Saving is **debounced**: calling `save()` writes to localStorage immediately in guest mode, while cloud mode calls `flushSave()` after 2 seconds to run Firestore `setDoc(..., {merge:true})`. Saves are also force-flushed on tab hide/`pagehide`.

### Core Functions

| Function | Role |
|---|---|
| `normalize(s)` | Normalizes loaded raw data into a safe schema (fills missing fields, validates currency) |
| `ruleHitsDate(rule, d)` | Determines whether a recurring item occurs on a given date (weekly/monthly/yearly + start/end dates + end-of-month correction) |
| `entriesForDate(dk)` | Returns that day's regular transactions plus recurring occurrences, combined |
| `aggregate(from, to)` | Period aggregation — income/expense totals, per-category totals, per-day totals |
| `toKRW(amount, cur)` | Converts an amount in a currency to KRW (using the rate table `rateKRW`) |
| `wonShort()` / `fmtCur()` | Amount abbreviation and currency-symbol formatting (per language) |

---

## 🗂️ Data

All data lives in a single `state` object. `freshState()` supplies the initial values and `normalize()` enforces the schema.

```js
state = {
  nick:    "나의 장부",     // display nickname (default "나의 장부" = "My Ledger"; Google displayName when signed in)
  created: "2026-07-01",    // creation date (YYYY-MM-DD)
  tx:        [ …transactions… ],    // regular transaction array
  recurring: [ …recurring… ],    // fixed recurring rule array
  cats:      { in:[…], out:[…] }  // category definitions (defaults + user-added)
}
```

### Transactions `state.tx[]`

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique id (`Date.now().toString(36)+random`) |
| `date` | string | Date `YYYY-MM-DD` |
| `type` | `"in"` \| `"out"` | Income / expense |
| `cur` | `"KRW"`\|`"USD"`\|`"AUD"`\|`"PHP"` | Currency |
| `amount` | number | Amount (integer for KRW, 2 decimal places for foreign currencies) |
| `cat` | string | Category id (e.g. `food`, `salary`) |
| `memo` | string | Memo (optional) |

```jsonc
{ "id":"lx8q3f21", "date":"2026-07-01", "type":"out",
  "cur":"KRW", "amount":8500, "cat":"food", "memo":"점심" }  // memo "점심" = "lunch"
```

### Fixed recurring items `state.recurring[]`

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique id |
| `type` | `"in"` \| `"out"` | Income / expense |
| `cur` | Currency code | Currency |
| `amount` | number | Amount |
| `cat` | string | Category id |
| `memo` | string | Name/memo (e.g. Netflix) |
| `freq` | `"monthly"`\|`"weekly"`\|`"yearly"` | Cycle |
| `dom` | number(1–31) | "Day of month" for monthly/yearly (end-of-month correction: day 31 → Feb 28/29) |
| `dow` | number(0–6) | Day of week for weekly (0 = Sunday) |
| `mon` | number(1–12) | "Month" for yearly |
| `start` / `end` | string | Effective start/end dates (optional, `YYYY-MM-DD`) |

```jsonc
{ "id":"m2k9d0aa", "type":"in", "cur":"KRW", "amount":3200000,
  "cat":"salary", "memo":"월급", "freq":"monthly", "dom":25,
  "start":"", "end":"" }  // memo "월급" = "salary"
```

### Categories `state.cats`

Default categories are hardcoded (10 expense + 5 income), and when the user adds a new category via "＋" in the sheet, it is appended to `state.cats[type]`.

| Group | Default categories (id · emoji · Korean) |
|---|---|
| Expense (`out`) | 식비🍚 (food) · 카페·간식☕ (cafe/snacks) · 교통🚌 (transport) · 쇼핑🛍️ (shopping) · 생활🏠 (living) · 의료·건강💊 (medical/health) · 문화·여가🎮 (culture/leisure) · 구독📺 (subscriptions) · 적금·저축🏦 (savings) · 기타💸 (other) |
| Income (`in`) | 월급💰 (salary) · 용돈🎁 (allowance) · 부수입💼 (side income) · 이자·배당🪙 (interest/dividends) · 기타➕ (other) |

---

## 💾 Storage / DB

The app has two storage modes (`mode` = `"cloud"` or `"guest"`).

### 1) Cloud — Firebase Firestore

- **Project:** `haru-221ae` (it **reuses** sibling app `haru`'s Firebase project, separating only the collection)
- **Collection / document:** `jangbu/{uid}` — one document per user holding the entire `state`
- **Auth:** Google sign-in (`GoogleAuthProvider`), popup first → redirect fallback if blocked
- **Offline:** `persistentLocalCache` (single tab) + `experimentalForceLongPolling` for offline/cache-first loading
- **Web apiKey:** The `AIza…` value exposed in the code is a **public web config value** (not a password). Actual access control is enforced by the Firestore security rules below.

**Required Firestore security rules:**

```
match /jangbu/{uid} {
  allow read, write: if request.auth != null && request.auth.uid == uid;
}
```

> Without these rules, cloud saving does not work, so until they are in place the app shows a notice recommending "this-device-only (guest)" use.

### 2) Guest / offline fallback — localStorage

When used without signing in, all data is stored only in the browser's `localStorage`.

| Key | Purpose |
|---|---|
| `jangbu-guest` | The entire `state` in guest mode (JSON) |
| `jangbu-mode` | Last-used mode (`"guest"` / `"cloud"`) — auto-restored on return visits |
| `jangbu-theme` | Theme (`"light"` / `"dark"`) — read first by an inline `<head>` script to prevent FOUC |
| `jangbu-lang` | Language (`"ko"` / `"en"`) |
| `jangbu-rates` | Exchange-rate cache (`{rateKRW, meta}`) — cached for 12 hours |

Backup: Settings → "Export data" saves `state` to a `jangbu-backup-YYYY-MM-DD.json` file.

---

## 🌐 External APIs · Dependencies

| Service | Purpose | Key required | Where it goes |
|---|---|---|---|
| **Firebase** (App/Auth/Firestore) SDK 12.14.0 | Auth and cloud storage | Web config values (public) | Code constant `FIREBASE_CONFIG` |
| **open.er-api.com** `/v6/latest/USD` | Live exchange rates (primary) | ❌ No key needed (free, CORS-enabled) | N/A |
| **jsdelivr currency-api** (`@fawazahmed0/currency-api`) `/v1/currencies/usd.json` | Exchange rates (secondary fallback) | ❌ Not needed | N/A |

**Exchange-rate logic:** The base currency is the **Korean won (KRW)**. `rateKRW = {KRW:1, USD, AUD, PHP}` (KRW per 1 unit of each currency). When the API returns USD-based rates, they are converted to KRW-equivalent values, stored, and cached in `localStorage["jangbu-rates"]`. If the cache is under 12 hours old, no new request is made; if both APIs fail, it falls back to hardcoded approximations (`USD≈1380, AUD≈900, PHP≈24`). Rates can be force-refreshed via "환율 새로고침" (refresh exchange rates) on the settings screen.

> Note: Kakao, TravelTime, Google, Skyscanner, Web Speech, and other services found in the `haru`/`moa` sibling apps are **not used in this app.** The only external calls are Firebase and the exchange-rate APIs.

---

## ▶️ Running Locally

Since there is no `package.json` or build step, simply serve it with any static file server. (Because of ES module imports, a local server is recommended over opening via `file://` directly.)

```bash
# Use any static server from the repository folder
python -m http.server 8000
#  → open http://localhost:8000

# Or, if Node is available
npx serve .
```

> Google sign-in only works on authorized domains (`localhost`, the GitHub Pages domain). If sign-in is blocked locally, all features are still available via "this-device-only without signing in (guest)".

---

## 🚀 Deployment

Deployed as static hosting on **GitHub Pages** (no build step).

1. In the repository **Settings → Pages**, set the source branch to `main` (root `/`).
2. Push `index.html` and it deploys automatically.
3. Result: https://clayborneyeounjunlee.github.io/jangbu/
4. Add the Pages domain under **Authentication → Authorized domains** in the Firebase console for Google sign-in to work.

> Firebase Hosting is not used, and the repo has no `firebase.json`/`.firebaserc`. Firebase serves only as the Auth + Firestore backend.

---

## 📁 File Structure

```
jangbu/
└── index.html   # the entire app — markup + CSS + JS (ES modules) in one file
```

Logical layout inside `index.html` (by comment sections):

```
<head>
 ├─ theme FOUC-prevention inline script (pre-applies localStorage["jangbu-theme"])
 └─ <style> CSS-variable-based light/dark themes; calendar/sheet/tab-bar styles
<body>
 ├─ #screen-loading / #screen-auth / #app  (3 top-level screens)
 ├─ 4 tab screens (calendar/recurring/analysis/settings) + FAB + bottom tab bar
 ├─ #sheet-wrap (bottom modal sheet) · #toast
 └─ <script type="module">
     ├─ FIREBASE_CONFIG / initFirebase()      dynamic Firebase loading
     ├─ utilities (date keys, escaping, amount formatting)
     ├─ currency/exchange rates (CURS, rateKRW, fetchRates, toKRW)
     ├─ i18n (T = {ko, en}, tr())
     ├─ categories (DEFAULT_CATS)
     ├─ state/saving (state, normalize, save/flushSave, Firestore setDoc)
     ├─ recurring-rule computation (ruleHitsDate, entriesForDate, aggregate)
     ├─ renderers (renderCalendar / renderRecurring / renderAnalysis / renderProfile)
     ├─ sheets (transaction/recurring/day detail) · toast · FAB
     ├─ theme / language toggles
     └─ sign-in (Google/guest) · init boot IIFE
```

---

## 🔗 Related Apps (moa collection)

- **Moa hub:** https://clayborneyeounjunlee.github.io/moa/ — reachable via the ◈ button at the top right of the app
- This app is the household-ledger version built by cloning sibling app **haru**'s wiring (auth, storage, theme, and i18n structure), sharing the Firebase project (`haru-221ae`) while separating the Firestore collection into `jangbu`.

---

<p align="center"><sub>Jangbu · 장부 · moa collection · brand color <b>#7c5cff</b> (violet)</sub></p>
