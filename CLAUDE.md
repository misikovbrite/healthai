# Longevity Blueprint — HealthAI App

## Overview
Single-file health & longevity coaching PWA. All HTML/CSS/JS in one `index.html` (~310KB).

## Deploy
```bash
curl --ftp-ssl --insecure -u "claudemain:mA2oR8sB1s" -T index.html "ftp://185.251.91.174:21/healthai/index.html"
```

## Critical: Temporal Dead Zone (TDZ) Pattern
`init()` is called early (~line 734) but many variables are declared after it. Any variable referenced during init() execution MUST use `var` (not `let`/`const`).

Known `var` variables: `U`, `DONE`, `CAL`, `OB`, `oi`, `obBuilt`, `goalIcons`, `evData`, `slideAnimating`, `curDay`, `curDateKey`, `curDayIdx`, `kbIndex`, `aiBuiltFlag`.

**Rule**: Before adding a new variable, check if it's used in init() → buildApp() → rob() call chain. If yes → use `var`.

## Architecture

### Storage
- `LS` wrapper: `{get(k), set(k,v), del(k)}` over localStorage (sync)
- Keys: `longevity-user-data`, `longevity_done_v2`, `lb-theme`, `longevity_ai_history`

### CSS (~lines 15-692)
- Custom properties in `:root` (dark) and `[data-theme="light"]`
- Short color vars: `--g` green, `--r` red, `--b` blue, `--p` purple, `--o` orange, `--c` cyan, `--pk` pink
- Spring easing: `cubic-bezier(.16,1,.3,1)` used everywhere

### Content System (~lines 1347-1944)
- `getP()` returns object with 18 sections, ~141 items total
- Item shape: `{ic, bg, c, t, s, badge, b}` where `b` is HTML body
- Short property names: `t`=title, `s`=subtitle, `n`=name, `d`=description, `dr`=duration, `c`=color, `bg`=background, `ic`=icon
- Evidence: `ev([{t:'finding', s:'source'}])` creates evidence buttons
- `detectLevel(src)` scores evidence quality (meta-analysis=95, RCT=90, cohort=75, etc.)

### AI Chat (~lines 2504-2650)
- GPT-4o-mini via OpenAI API, streaming responses
- **RAG-lite**: `searchKB(query, 3)` finds top 3 relevant content items by keyword matching
- `buildKBIndex()` builds search index from getP() on first AI chat open
- System prompt includes user profile + relevant KB articles (saves ~98% tokens vs full KB)

### Key Functions
- `init()` → entry point, calls buildApp()
- `buildApp()` → builds Today page (header, streak, ring, calendar, timeline)
- `buildPlan()` → builds Explore page (card grid + accordion list)
- `buildAI()` → builds AI chat page
- `rob()` → renders onboarding flow
- `buildCAL()` → converts CAL_BASE offsets to actual times using U.wakeTime
- `nav(pg)` → page navigation (pg-today, pg-explore, pg-ai)

### Schedule System
- `CAL_BASE`: 7-day schedule with time offsets from wake/bed time
- `buildCAL()` converts offsets to real times
- Timeline shows events for current day with completion tracking
