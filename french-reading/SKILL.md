---
name: french-reading
description: Generate an interactive French reading learning page (React JSX artifact) from a French article or PDF. Use when the user provides a French article, text, or PDF and asks to create a learning page, study page, or lecture interactive. Extracts B2/C1 vocabulary with lemma-based dictionary entries, grammar analysis, translation buttons, writing practice, and spaced repetition flashcards.
---

# French Reading Interactive Page Generator

When the user provides a French article (text or PDF), generate a complete React JSX file with all article data baked in as a static constant. Output as a `.jsx` file to `/mnt/user-data/outputs/`. No file import/upload UI needed inside the artifact.

---

## STEP 1 — Extract ARTICLE data object

```js
const ARTICLE = {
  title: "Article title",
  mode: "article",           // always "article" unless it's a spoken transcript
  paragraphs: ["§1 text", "§2 text", ...],
  vocab: [ /* 25–40 items, see rules below */ ],
  grammar: [ /* 5–6 items, see rules below */ ]
};
```

### Paragraphs
- Split at natural paragraph breaks (blank lines, topic shifts)
- Preserve ALL French accents and punctuation exactly
- Use backtick template literals for strings that contain `"` double quotes

### Vocabulary — 25–40 items
Only B2 and C1 words. Skip A1–B1 basics.

| Field | Content |
|-------|---------|
| `lemma` | Dictionary form (infinitive / m.sg. / sg.) |
| `pos` | `n.m.`, `n.f.`, `v.`, `v. pron.`, `adj.`, `adv.`, `loc. adv.`, `loc. v.`, `loc. conj.` … |
| `level` | `"B2"` or `"C1"` |
| `cn` | Chinese definition (simplified) |
| `example` | One natural French example sentence |
| `forms` | Array of exact surface forms as they appear in the article |

- Deduplicate by `lemma`
- Multi-word expressions (e.g. `lâcher prise`, `en filigrane`) are valid entries

### Grammar — 5–6 items
Pick complex structures worth deep analysis (relative pronouns, participial clauses, Latin adverbs, inversion, causative rendre, paradox structures). For poetry, also include rhetorical devices (personnification, allitération, métaphore).

```js
{
  text: "exact phrase from article (max ~80 chars)",
  analysis: `【Structure】abbreviated grammar pattern with **bold** markers

【Analyse】Chinese explanation with *French examples in italic* and **key terms bolded**:
- list item one
- list item two

【Note】usage notes or comparisons:
- *example 1*（中文说明）
- *example 2*（中文说明）`
}
```

- `text` must be a substring of one of the paragraphs (used for matching). For poetry, the paragraphs use `\n` as line separators — the match normalizer in the JSX handles both `\n` and `/` as separators, but keep `text` in either format.
- Use linguistic abbreviations in 【Structure】: `v.`, `adj.`, `inf.`, `p.p.`, `prop.`, `COD`, `gérondif`, `loc. adv.` etc. — NOT full Chinese grammar terms
- 【Analyse】and 【Note】in Chinese
- **Use markdown formatting inside analysis**: `**bold**` for key terms/structures, `*italic*` for French words/examples, `- ` for list items. Separate paragraphs with blank lines. The JSX renderer (`GrammarAnalysis` component) parses these markers and renders styled output.

---

## STEP 2 — Full JSX Component

Single-file React component. All data baked in. Uses hooks from React. Default export.

### Imports & setup

```jsx
import { useState, useEffect, useRef, useCallback } from "react";
```

Load Google Fonts via `<link>` inside JSX:
```
Crimson Pro (French text) · DM Sans (UI) · Noto Sans SC (Chinese)
```

### Storage helpers (window.storage with localStorage fallback)

```js
async function storageGet(key) {
  try { const r = await window.storage.get(key); return r?.value ? JSON.parse(r.value) : null; }
  catch { try { return JSON.parse(localStorage.getItem(key)); } catch { return null; } }
}
async function storageSet(key, value) {
  const str = JSON.stringify(value);
  try { await window.storage.set(key, str); } catch { try { localStorage.setItem(key, str); } catch {} }
}
```

Storage keys (all per-article, using slug — NO cross-article accumulation):
- `"vocab-[slug]"` — vocabulary array for this article only (article vocab + user-added words)
- `"sr-[slug]"` — spaced repetition progress for this article `{ [lemma]: { level, nextReview } }`
- `"writing-[slug]"` — user writing per paragraph `{ [idx]: string }`
- `"feedback-[slug]"` — AI feedback per paragraph `{ [idx]: string }`

Slug = `ARTICLE.title.toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/(^-|-$)/g, "")`

### Claude API helper

```js
async function callClaude(prompt, system = "") {
  const body = {
    model: "claude-sonnet-4-20250514",
    max_tokens: 1000,
    messages: [{ role: "user", content: prompt }]
  };
  if (system) body.system = system;
  const res = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body)
  });
  const data = await res.json();
  return data.content?.map(b => b.text || "").join("") || "";
}
```

Prompts:
- **Translation**: `"Translate into Chinese. ONLY output translation.\n\n[paragraph text]"`
- **Writing correction**: `"French writing coach. Student imitates:\n\"[original]\"\n\nStudent wrote:\n\"[text]\"\n\nFeedback in Chinese: 1.🟢Good 2.🔴Errors 3.🟡Suggestions 4.✍️Corrected. Be encouraging."`
- **Word lookup**: `"For the French word \"[word]\" in context \"[sentence]\", provide JSON: {\"lemma\":\"...\",\"pos\":\"...\",\"level\":\"A1/A2/B1/B2/C1/C2\",\"cn\":\"...\",\"example\":\"...\"} ONLY output JSON."`

### Spaced repetition (Ebbinghaus, 9 levels)

```js
const SR_INTERVALS = [60000, 600000, 3600000, 28800000, 86400000, 259200000, 604800000, 1296000000, 2592000000];
// 1min · 10min · 1h · 8h · 1j · 3j · 7j · 15j · 30j

function srStatus(card) {
  if (!card) return "new";
  if (card.level >= SR_INTERVALS.length - 1) return "mastered";
  if (card.nextReview <= Date.now()) return "due";
  return "learning";
}
function srLabel(ms) {
  if (ms < 3600000) return `${Math.round(ms/60000)}min`;
  if (ms < 86400000) return `${Math.round(ms/3600000)}h`;
  return `${Math.round(ms/86400000)}j`;
}
```

- Remember → `level + 1` (max 8)
- Forget → `level - 2` (min 0)
- **Both actions always advance to the next card** (`setFcIndex(i => i + 1)`) — do NOT gate advancement behind `if (remembered)`

### Stable card list (anti-flicker)

**Root cause of flicker**: if `fcCards` is recomputed on every render (via inline filter or `useMemo`/`useCallback` depending on `srProgress`), then `srStatus()` calling `Date.now()` can shift cards between categories mid-render, and `fcIndex % newLength` jumps to a different card.

**Solution**: `fcCards` is a `useState([])`, rebuilt only on explicit triggers:

```js
const [fcCards, setFcCards] = useState([]);
const buildCards = useCallback((vocab, sr, filter) => {
  return vocab.filter(v => { /* srStatus filtering */ });
}, []);

// Rebuild on vocab load or filter change — NOT on srProgress change
useEffect(() => {
  if (accVocab.length > 0) setFcCards(buildCards(accVocab, srProgress, fcFilter));
}, [accVocab, fcFilter]);

// In srAct: rebuild synchronously with the new SR data
const newCards = buildCards(accVocab, updatedSrProgress, fcFilter);
setFcCards(newCards);
```

**Anti-flicker on SR action**: use `srAnimating` state — set `true` before updates, reset after 2 `requestAnimationFrame` ticks. Card container uses `transition: srAnimating ? "none" : "transform 0.4s ease"` to suppress the flip animation during card switch.

### Word tokenizer

```js
function tokenize(text) {
  const parts = [];
  const regex = /([a-zA-ZÀ-öø-ÿ'-]+|[^a-zA-ZÀ-öø-ÿ'-]+)/g;
  let m;
  while ((m = regex.exec(text)) !== null) parts.push(m[0]);
  return parts;
}
```

---

## STEP 3 — UI Structure

### Color tokens
```
Background:   #FAF8F5
Card:         #FFFFFF
Warm:         #F5F0EB
Accent:       #C85A3A  (terracotta — header, badges, primary buttons)
B2 word:      color #3A7CA5 / bg #E8F2F8
C1 word:      color #8B5E3C / bg #F5EDE5
Green:        #4A8C6F  (AI correction button, SR "acquis")
Yellow:       #C4960C  (learning filter)
```

### Layout
```
HEADER (terracotta bar)
  mode badge · title · stats (vocab count · grammar count · § count)

TABS
  📖 Lecture | 🗂️ Flashcards

VOCAB LEGEND (reading tab only)
  B2 chip · C1 chip · hint text
```

### Tab 1 — 📖 Lecture

Per paragraph (`§ N` badge):

1. **Article text** — `HighlightedText` component
   - All tokens rendered via `tokenize()`
   - B2/C1 words: highlighted, clickable → vocab popup
   - Any other word: clickable → word lookup dialog (hover bg `#f0ede8`)
   - Forms matching: `formsMap` built from `vocab[].forms` (lowercased)

2. **Translation panel** (shown when fetched, togglable hide/show)
   - bg `#F5F0EB`, Noto Sans SC, 14px

3. **Grammar cards** (expandable, matched by substring to paragraph)
   - Collapsed header: italic phrase + ▼/▲
   - Expanded: rendered by `GrammarAnalysis` component (see below) — **NOT** plain `pre-wrap`
   - Match using normalizer: `s.replace(/[\s\/\n]+/g, " ").trim().toLowerCase()` to handle poetry (grammar.text with `/`) and prose (paragraphs with `\n`) uniformly

**`GrammarAnalysis` component** — required for readable grammar rendering:
- Parses `【Structure】【Analyse】【Note】` sections and renders each with a colored pill title (blue / terracotta / green respectively)
- Sections separated by dashed horizontal lines
- Inline markdown: `**bold**` → `<strong>` with dark weight; `*italic*` → serif italic in warm brown (`#8B5E3C`) for French examples
- `- ` at line start → bulleted `<ul>` list with disc markers
- Blank lines separate paragraphs
- Font: Noto Sans SC 13.5px for Chinese base, Crimson Pro for italic French
- This component is essential. Without it, the analysis text is dense and hard to scan; with it, the reading experience matches the clarity of a Markdown-rendered chat message.

4. **Action row**: Traduire / Cacher / Afficher button

5. **Writing practice**
   - Label: `✍️ IMITATION — Réécrivez ce paragraphe avec vos propres mots`
   - `<textarea>` auto-saves on change to storage
   - "🤖 Corriger" button (green) → calls Claude API → shows feedback
   - Feedback panel: bg `#F0F7F4`, Noto Sans SC

### Tab 2 — 🗂️ Flashcards

**Stats/filter bar** (5 pills):
`Tout (N)` · `À revoir (due+new)` · `Nouveau (N)` · `En cours (N)` · `Acquis (N)`

Active pill: filled with pill's color. Inactive: outlined.

**Flashcard (3D flip)**:
```css
perspective: 1000px
transformStyle: preserve-3d
transform: rotateY(0deg) → rotateY(180deg)
transition: 0.4s ease
backfaceVisibility: hidden
```

Front face:
- `fromPrev` flag → `📚 Article précédent` badge
- `userAdded` flag → `⭐ Ajouté manuellement` badge
- Lemma (Crimson Pro 28px bold)
- `pos · LEVEL` (colored chip)
- "Cliquez ou Espace pour retourner" hint
- 9 SR progress dots (green = reached, gray = not yet)

Back face:
- Lemma (small terracotta)
- Chinese definition (Noto Sans SC 20px)
- Example (italic, Crimson Pro)
- Forms if surface ≠ lemma
- Two SR buttons:
  - `← Revoir · [interval]` (terracotta)
  - `Acquis → [interval]` (green)

**Keyboard controls** (useEffect on `tab === "flashcards"`):
- Space / Enter → flip
- ← → Revoir (forget)
- → → Acquis (remember)

**Counter**: `X / N · ← Oublié · → Acquis · Espace = retourner`

**Bottom — Full vocabulary list**:
Below the flashcard area, render the complete `accVocab` array as a scrollable list. Each item shows:
- SR status dot (green=mastered, yellow=learning, terracotta=due, gray=new)
- Lemma (Crimson Pro 17px bold) + pos·level chip
- `userAdded` → ⭐ badge
- Chinese definition (Noto Sans SC 13px)
- Example sentence (italic Crimson Pro 13px)
- Forms if surface ≠ lemma
No export vocab button — the list IS the reference.

### Vocab popup (modal overlay)
Triggered by clicking a highlighted word. Shows: lemma, pos, level chip, Chinese def, example, forms (if ≠ lemma). "Fermer" button. Click outside to close.

### Word lookup dialog (modal overlay)
Triggered by clicking a non-highlighted word. Shows the word in `«  »` quotes. Two buttons:
- `🔍 Chercher et ajouter` → calls Claude API → adds to `accVocab` + saves to storage → shows toast `✅ Ajouté aux flashcards`
- `✕` cancel

### Toast notification
Fixed bottom center, dark bg, 2500ms auto-dismiss.

---

## STEP 4 — Per-article vocabulary logic (NO cross-article accumulation)

On mount (`useEffect`):
1. Start with `ARTICLE.vocab` as the base vocabulary
2. Load `vocab-[slug]` from storage — if it contains user-added words (`userAdded: true`), append them
3. Save the combined list back to `vocab-[slug]`
4. Load `sr-[slug]` SR progress

Each article's vocab and SR progress are fully independent. No `fromPrev` merging, no global keys.

User-added words (from word lookup) get `userAdded: true` flag, appended to `accVocab` + saved to `vocab-[slug]` immediately.

---

## STEP 5 — Critical implementation rules

1. **Emoji**: always literal characters in JSX strings — NEVER `\uXXXX` escape sequences
2. **Strings with double quotes**: use backtick template literals or single-quoted strings
3. **No `<form>` tags** anywhere — use `onClick`/`onChange` handlers only
4. **Never use localStorage directly** — always use `storageGet`/`storageSet` helpers (which fall back to localStorage internally)
5. **Grammar matching**: use `para.includes(g.text)` — `g.text` must be an exact substring of a paragraph
6. **SR button clicks**: always call `e.stopPropagation()` to prevent card flip re-trigger
7. **`setFcFlipped(false)`** after every SR action
8. **`setFcIndex(i => i + 1)`** after every SR action (both remember AND forget) — never use `if (remembered)` guard
9. **`fcCards` must be a `useState`**, NOT computed inline or via `useMemo`/`useCallback` with `srProgress` dependency. Rebuild only on: (a) vocab load, (b) filter change, (c) synchronously inside `srAct` with the new SR data. This prevents `Date.now()` in `srStatus()` from causing mid-render list shifts.
10. **Anti-flicker on SR**: use `srAnimating` state — set `true` before updates, reset after 2 `requestAnimationFrame` ticks; card container uses `transition: srAnimating ? "none" : "transform 0.4s ease"`
11. **`fcIndex % Math.max(cards.length, 1)`** to avoid division-by-zero
12. **No vocab export button** — render full vocabulary list inline below flashcard area instead

---

## OUTPUT

Generate the complete `.jsx` file as a single artifact saved to `/mnt/user-data/outputs/[article-slug]-lecture-interactive.jsx`. Present it with `present_files`. All article data baked in. No file import UI.
