# Spiskammer & Gjøremål — Project Journal

A complete record of building two phone-only PWAs from scratch, entirely from a phone (no laptop), over several weeks of conversation with Claude.

---

## Table of contents

1. [Origin story](#origin-story)
2. [The phone-only constraint](#the-phone-only-constraint)
3. [Spiskammer — the pantry & shopping app](#spiskammer)
4. [Gjøremål — the to-do app spin-off](#gjøremål)
5. [Lessons & gotchas](#lessons--gotchas)
6. [Reference: data model](#reference-data-model)
7. [Reference: AI prompts](#reference-ai-prompts)
8. [Reference: features deferred for future builds](#deferred-features)

---

## Origin story

Started with a photo of HelloFresh leftovers on a kitchen table. Tom asked Claude to identify the ingredients in the photo, then to build an "interactive virtual pantry" to track them. Claude built a React artifact with categorised ingredients, +/- quantity controls, and checkboxes.

Conversation evolved from "can you make this an Android app?" into a phone-only PWA deployment, building everything live from a Samsung S24 — no laptop access. The app then grew over multiple iterations into a polished tool with shopping lists, AI integration, and a custom aesthetic — then spawned a parallel to-do app reusing the same architecture.

---

## The phone-only constraint

The biggest design constraint throughout: **everything was deployed, debugged, and iterated from a phone**. No code editor, no terminal access, no laptop. This shaped many decisions:

- **GitHub web UI for all uploads** — no `git push`, no CLI. Files uploaded via tap-tap-tap on github.com.
- **GitHub Pages free hosting** — no servers to set up, no CI/CD config.
- **Single-file PWAs** — React, Tailwind, and Babel loaded from CDN, no build step. Edit one HTML file, upload, done.
- **Conservative JavaScript syntax** — early bugs from arrow functions and template literals in Babel-in-browser led to using `function() {}` declarations throughout. More verbose but rock-solid.
- **Diagnostics by description** — when something broke, Tom described what he saw or sent screenshots. No console access. Several issues traced this way: blank-screen-on-update was a service worker cache problem; "downloads index.html" was Pages not yet deployed; "import fails in Brave but works in Firefox" was a browser file-picker quirk.

---

## Spiskammer

**Repository**: `https://autominablesnowbot.github.io/pantry/`

Norwegian for "pantry/larder." A PWA for tracking what's in your kitchen and what you need to buy.

### Evolution by version

#### v1 — proof of concept
- React artifact in chat with hardcoded ingredient list from the original HelloFresh photo
- +/- quantity controls, search, hide-empty filter
- Five categories: Canned & Boxed, Grains & Legumes, Spices & Seasonings, Sauces & Wet, Fresh / Other

#### v2 — deployed as PWA
- Wrapped as a Progressive Web App: `index.html`, `manifest.json`, `sw.js`, two PNG icons
- localStorage for persistence
- Service worker for offline use
- Deployed to GitHub Pages from phone via web UI
- Add-to-Home-Screen on Android works as a real installed app

Added soon after:
- **Import button** — load JSON files with new items
- **The Brave browser caching problem** — fixed by clearing site data, taught us to bump service worker cache version on every deploy

#### v3 — staples and shopping list
Major redesign. Two tabs in a single app:
- **Pantry** — stock tracking
- **Shopping** — what to buy

Key concepts introduced:
- **Staples** (⭐): items you always want in stock (oil, salt, eggs). Marked with a star, moved to a dedicated section, no quantity tracking — just a "Have / Need" toggle. When toggled to Need, they auto-appear on the shopping list.
- **Quick-add grid** — historically-ranked tiles for fast adding to shopping list
- **Autocomplete** from history
- **Two removal paths** on shopping list: ✓ moves to pantry, 🗑 just removes

Underlying data model went from one localStorage key to four: items, shopping list, history, and view state.

#### v4 — multiple lists, voice, AI prompt, retheme

A lot happened here.

**Renamed to Spiskammer**. Updated title, manifest, header.

**Removed the green "used" tick** on pantry items (redundant with quantity counter) and replaced with a 🛒 button that adds items directly to the main shopping list (greys out if already there).

**Multiple shopping lists**:
- Main list (always present, can be renamed, can't be deleted) gets privileged behaviours: staples auto-populate here, quick-add grid uses history, pantry 🛒 sends here
- Extra lists (IKEA, hardware, etc.) — simpler, no grid, no auto-populate, manual add only
- Long-press a list pill to rename/delete

**Always-buy items** (📌):
- Distinct from staples — for things you always need to *buy* regardless of stock at home (the canonical example was Tom's wife's liquorice)
- Long-press a Main list item → Mark as always-buy
- Pin badge, sorts to top
- Reappear automatically when you clear the shopping list
- Only on Main list

**Voice input** (🎤):
- Web Speech API, fully on-device, no API key
- Hidden in browsers that don't support it (so Firefox doesn't see the button)
- Light regex parsing: "2 onions", "two onions" → qty 2
- Speak naturally with commas/and as separators

**AI Helper**:
- A copy-to-clipboard prompt for any AI chat
- Returns JSON in a format Spiskammer can import
- Initial design was food-only; later expanded to handle anything (IKEA shopping, hardware, etc.) by returning empty category for non-food items
- One contradiction caught in testing: early prompt said both "output JSON" and "save as file" — AIs got confused. Fixed by removing the file-save instruction.

**Open to last-used tab** — remembers where you were.

**Migration on first load** — v3 data converted automatically.

#### v4 retheme — warm jars & shelf aesthetic

Tom uploaded an AI-generated icon (jars of pasta on a wooden shelf with sage leaves) and asked for the app to match. Did a full retheme:

- Background: cream/parchment instead of orange
- Primary action colour: warm wood brown then later olive green for shopping
- Serif heading font: Cormorant Garamond from Google Fonts
- Header icon became the actual artwork (large rounded image)
- Category colours muted to harmonise
- Bigger ✓ button vs smaller 🗑 to reduce mid-shop fat-finger risk
- AI Helper became a stone-toned card with a robot avatar

Tom also uploaded a polished mockup with bottom navigation, per-item product icons, and a bell — Claude flagged these as needing separate conversations and stuck to "easy CSS stuff only" per Tom's instruction.

#### v5+ — refinements after using v3 in anger

Tom used v3 for a week then came back with a punch-list. Each got built:

- **Paste JSON button** — solves the "Brave can't pick up file uploads" problem and the "AI returns JSON in chat, not as a file" problem. Tap, paste, parse. Auto-fills from clipboard when possible.
- **Always-buy items don't show the green tick** — only the trash. Trash gets bigger to fill the space.
- **Quick-add collapsible**, defaults to collapsed, auto-expands when typing
- **Header compacted** — icon shrank, title smaller, "X on list" badge moved inline, tab buttons tightened
- **AI Helper as button-with-modal** instead of always-visible card
- **One merged card** — list pills, add area, quick-add, and the list itself all share one outer card with faint dividers between sections

#### Things logged for next time (v5 features doc)

- Quick-add grid should rank by *shopping* history, not all history (rarely-bought pantry items shouldn't outrank weekly milk)
- In-app feedback button (mailto with pre-filled debug info) — deferred until tester count grows beyond 2

---

## Gjøremål

**Description**: A to-do app spun off from Spiskammer's bones.

Tom realised he was already using Spiskammer as a quasi-todo list, and asked for a dedicated version. The instruction was "remove a lot of functionality" — keep the good parts of the architecture, strip pantry-specific stuff.

### What was kept

- The merged-card layout
- Serif title, warm palette (shifted to blue accent instead of olive green)
- Multiple lists with pills (Main + extras)
- Add input + Import + Paste + Voice + AI in one row
- Voice input with sentence splitting
- AI Helper with a to-do-specific prompt
- Long-press menu for item actions
- Offline PWA, GitHub Pages deploy
- Same conservative JS style

### What was removed

- Pantry tab entirely
- Staples concept (a to-do isn't something you "stock")
- Quantity counters (a task is one task)
- Categories and colour-coding
- Quick-add grid (todos don't repeat the way groceries do)
- Always-buy items
- "Move to pantry" logic

### What was added

- **Due dates** on tasks (optional, set via long-press menu)
- **Smart date formatting**: "Today", "Tomorrow", "In 3d", "5d overdue", or a "12 Jun" style for further out
- **Colour-coded urgency**: red for overdue, amber for today, yellow for soon, stone for further out
- **Mark done without delete** — completed tasks can stay visible (struck-through) so you have a record
- **Show/hide completed** toggle, persisted
- **Clear completed** action at the bottom
- **Sorting** — incomplete first by due date, then complete by recency

### Storage keys

- `gjoremal-lists`
- `gjoremal-active-list`
- `gjoremal-show-completed`

### Icon

Hand-coded with PIL: a notepad on a cream background with three checklist lines — one ticked with a blue tick, two unticked. Matches Spiskammer's warm aesthetic but distinct enough to tell apart on a home screen.

---

## Lessons & gotchas

### Browser compatibility

| Feature | Chrome Android | Firefox Android | Brave Android | Safari iOS |
|---------|----------------|-----------------|---------------|------------|
| Core PWA | ✓ | ✓ | ✓ | ✓ |
| Add to Home Screen | ✓ | ✓ | ✓ | ✓ (via Share) |
| Voice input | ✓ | ✗ (API exists but not wired up) | ✓ | unreliable |
| File picker for import | ✓ | ✓ | ✗ (hardened, can't access files) | ✓ |
| Clipboard paste | ✓ | ✓ | mostly | ✓ |
| localStorage persistence | ✓ | ✓ | ✓ | ⚠ may evict after 7 days |

**Bottom line**: Android Chrome is the best experience. Firefox is second-best (no voice). iOS works but watch out for storage eviction.

### Service worker caching

Every deploy needs a bumped `CACHE = 'name-vN'` string in `sw.js`, otherwise browsers serve old broken cached code. We hit this on every update; bumping the version forces clean re-fetch.

### Babel-in-browser limitations

The standalone Babel script that transpiles JSX in the browser is *less forgiving* than full Babel:
- Arrow functions sometimes fail in awkward positions
- Template literals in `className` props can break silently
- Escaped apostrophes inside JSX strings can be odd

Solution: write old-style `function() {}` everywhere, build className strings with `+` concatenation instead of template literals, use double quotes wherever possible.

### Mobile file pickers

Hardened browsers (Brave especially) sometimes can't pick up files from the file picker correctly — they pass a URI to a cloud file rather than the actual content. The Paste JSON workaround (paste text directly into a textarea) sidesteps this entirely.

### "Save as file" doesn't work for AIs on mobile

Asking an AI to "save as a downloadable file" is unreliable on mobile. Most chat UIs just print the JSON inline. Trying to force file output makes the AI hallucinate or wrap things in markdown. The clean solution: AI prints JSON in chat, user copies, app accepts paste.

### Service worker + GitHub Pages

GitHub Pages has a ~1-2 minute deploy delay after pushing. Combined with service worker caching, this means after every commit:
1. Wait 1-2 min
2. Force-refresh in browser (or rely on the cache version bump)
3. Sometimes wipe site data for stubborn cases

### The Quick-add grid wasn't quite right

Tom realised after a week's use: the grid ranks by *anything ever added* including pantry imports. A rarely-bought pantry item could outrank weekly milk. Logged for next build: separate `shopping-add-count` from total count, rank grid by the former.

---

## Reference: data model

### Spiskammer (v4+)

```
localStorage keys:
  spiskammer-items          → pantry items array
  spiskammer-lists          → shopping lists array (each has id, name, items, isMain, deletable)
  spiskammer-active-list    → id of currently active list
  spiskammer-history        → array of {key, name, addCount, lastAdded, lastCategory, lastUnit}
  spiskammer-always-buy     → array of always-buy items
  spiskammer-grid-expanded  → boolean
  spiskammer-last-view      → "pantry" or "shopping"
  spiskammer-main-cleared   → flag for repopulating always-buy items after a clear
  spiskammer-migrated       → migration flag

Pantry item shape:
  { id, name, qty, unit, category, staple, outOfStock }

Shopping list item shape:
  { id, name, qty, fromStaple, fromAlwaysBuy, suggestedCategory, suggestedUnit }

History entry shape:
  { key, name, addCount, lastAdded, lastCategory, lastUnit }

Always-buy item shape:
  { name, category, unit }
```

### Gjøremål (v1)

```
localStorage keys:
  gjoremal-lists           → array of lists
  gjoremal-active-list     → id of currently active list
  gjoremal-show-completed  → boolean

Task shape:
  { id, text, done, due, createdAt, completedAt }
```

---

## Reference: AI prompts

### Spiskammer (current — handles food and non-food)

```
You are an item extraction assistant. Analyse the provided input (photo, screenshot, or text) and return a JSON array of items.

RULES:
- Output ONLY a valid JSON array. No markdown code fences, no commentary, no preamble, no trailing text.
- Each item must have these exact fields: name, qty, unit, category.
- Use double quotes, not single quotes.

FIELD DEFINITIONS:
- name: Short, clear English name (e.g. "Red Kidney Beans", not "A can of red kidney beans")
- qty: Integer. Best guess from visual count or context. Default to 1.
- unit: Short human description like "390g box", "bunch", "whole", "pack". Empty string if unclear.
- category: Empty string "" for non-food items. For food items, MUST be one of:
  - "Canned & Boxed" — tinned goods, cartons, boxed shelf-stable items
  - "Grains & Legumes" — rice, pasta, lentils, beans (dried), flour, breadcrumbs, nuts
  - "Spices & Seasonings" — spices, dried herbs, stock powder, seasoning sachets
  - "Sauces & Wet" — oils, vinegars, soy sauce, mayo, yoghurt, tomato paste, honey
  - "Fresh / Other" — fresh produce, meat, fish, fresh herbs, dairy, eggs, bread

BEHAVIOUR:
- Merge duplicates: three of the same item = one entry with qty: 3
- If ambiguous, make your best guess and include it
- If nothing recognisable, return []
```

### Gjøremål

```
You are a task extraction assistant. Analyse the provided input (photo, screenshot, or text) and return a JSON array of to-do items.

RULES:
- Output ONLY a valid JSON array. No markdown code fences, no commentary, no preamble, no trailing text.
- Each item must have these fields: text (required), due (optional ISO date YYYY-MM-DD, empty string if none).
- Use double quotes, not single quotes.

FIELD DEFINITIONS:
- text: The task itself, written as a short imperative ("Call dentist", "Buy birthday card for mum"). Capitalise the first letter. Keep it under 80 characters.
- due: Optional ISO date if a deadline is mentioned ("by Friday", "next Tuesday"). Resolve relative dates against today. Empty string if no due date.

BEHAVIOUR:
- Each separate task gets its own entry. Don't merge unrelated tasks.
- Strip filler like "I need to", "Don't forget to", "Remember".
- If nothing actionable, return [].
```

---

## Deferred features

Captured in `FEATURES.md` in the Spiskammer repo for future builds.

### Spiskammer

- **Quick-add ranks by shopping history** (not all history)
- **In-app feedback button** — mailto with pre-filled debug info
- **Expiry tracking** — warnings as dates approach
- **Recipe suggestions** — "what can I cook with what's in stock?"
- **Barcode scanning** — camera-based item entry
- **Export/backup** — download all data as JSON
- **Meal planning** — assign items to planned meals, auto-deduct from pantry
- **Multi-location** — fridge vs freezer vs cupboard sub-locations
- **Sharing** — sync between household members (needs backend)
- **Bottom navigation** + bell + per-item product images (from the mockup, deferred)

### Gjøremål

(Initial v1 — no deferred features yet, awaiting Tom's feedback after a week of use.)

---

## File inventory

### Spiskammer repo

```
pantry/
├── index.html              # the whole app
├── sw.js                   # service worker
├── manifest.json           # PWA manifest
├── icon-192.png            # home screen icon (small)
├── icon-512.png            # home screen icon (large)
├── README.md               # repo description
└── FEATURES.md             # roadmap & queued features
```

### Gjøremål repo

```
gjoremal/
├── index.html
├── sw.js
├── manifest.json
├── icon-192.png
├── icon-512.png
└── README.md
```

### Generated artifacts

- `Spiskammer_User_Guide.pdf` — 7-page user guide with quick-start, full reference, troubleshooting, privacy notes
- `recipe-ingredients.json` — translated Norwegian recipe import sample
- `fridge-import.json` — first fridge contents import (one-time, from a photo)

---

## Workflow that worked

The pattern that worked best, repeatable for similar projects:

1. **Photo or description of what's wanted** → Claude builds initial artifact
2. **Iterate in chat** with the working artifact until it feels right
3. **Convert to PWA** when ready to deploy — single HTML file, plus manifest + sw + icons
4. **GitHub Pages deploy from phone** via web UI uploads
5. **Use in real life for a week** — surface real friction points
6. **Come back with a punch-list** of UI tweaks and small features
7. **Bump cache version, upload, refresh** — iteration loop is fast
8. **Log deferred ideas in FEATURES.md** in the repo so future-Claude can pick up the thread

A few "rules" that kept things working:

- **Don't break what works** — every iteration starts from the previous working version
- **Test in two browsers** — catches the Firefox vs Brave vs Chrome differences early
- **One change theme per session** — bundle related UI tweaks, then ship and use; don't try to land 12 unrelated changes at once
- **Capture the design before coding it** — especially for big features like Multiple Lists or the merged card layout, sketch the structure in words first
- **Keep the build script** — for the PDF, having `build_guide.py` in the conversation meant easy iteration

---

*End of journal.*
