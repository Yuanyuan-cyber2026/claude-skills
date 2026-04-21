# 📖 French Reading Skill for Claude

An interactive French reading learning page generator — paste any French article, get a full study tool with vocabulary, grammar analysis, and spaced repetition flashcards.

> Built for Chinese-speaking learners of French at B2–C1 level.
> 为以中文为母语、学习法语的B2–C1学习者设计。

---

## ✨ What it does

When you give Claude a French article (text or PDF), this skill generates a complete interactive learning page (React JSX) with:

| Feature | Description |
|---|---|
| 📄 **Annotated reading** | B2/C1 words highlighted and clickable in the article text |
| 🇨🇳 **Paragraph translation** | One-click Chinese translation via Claude API |
| 📚 **Vocabulary dictionary** | 25–40 B2/C1 words with lemma, POS, Chinese definition, example |
| 🔍 **Word lookup** | Click any unknown word → Claude adds it to your flashcards |
| 🧠 **Grammar analysis** | 5–6 complex structures explained in Chinese with examples |
| 🗂️ **Flashcards** | 3D flip cards with spaced repetition (Ebbinghaus, 9 levels) |
| ✍️ **Writing practice** | Imitation exercises with AI correction per paragraph |
| 💾 **Persistent progress** | SR progress saved per article via `window.storage` |

---

## 🚀 How to install

### Step 1 — Download the skill file

Download [`SKILL.md`](./french-reading/SKILL.md) from this repository.

### Step 2 — Upload to Claude.ai

1. Go to [claude.ai](https://claude.ai)
2. Open **Settings** → **Skills** (or **Profile**)
3. Click **Upload skill** and select the `SKILL.md` file

### Step 3 — That's it

Claude will now automatically use this skill whenever you send a French article and ask for a learning page.

---

## 💬 How to trigger it

Send a French article (paste the link, text or attach a PDF) with a prompt like:

```
Crée une page de lecture interactive pour cet article.
```

```
生成这篇法语文章的学习页面。
```

Claude will generate a `.jsx` file you can download and open locally with a React/Vite setup, or view directly in Claude's artifact panel.

---

## 🖼️ Features in detail

### Vocabulary (Tab: 📖 Lecture)
- B2 words highlighted in blue, C1 words in brown
- Click any highlighted word → popup with definition, example, and word forms
- Click any *other* word → Claude looks it up and adds it to your flashcards

### Flashcards (Tab: 🗂️ Flashcards)
- 3D flip card: front = French lemma, back = Chinese definition + example
- Spaced repetition: 9 levels from 1 min → 30 days (Ebbinghaus curve)
- Filter by: All / Due / New / Learning / Mastered
- Keyboard shortcuts: `Space` = flip, `←` = forgot, `→` = remembered
- Full vocabulary list shown below the card for reference

### Grammar analysis
- 5–6 complex structures per article
- Each entry: 【Structure】pattern + 【Analyse】Chinese explanation + 【Note】usage tips
- French examples in italic, key terms in bold

### Writing practice
- Per-paragraph imitation exercise
- AI correction in Chinese: ✅ strengths, ❌ errors, 💡 suggestions, ✍️ corrected version

---

## 🏗️ Technical architecture

- **Output**: single-file React JSX (all article data baked in as a static constant)
- **No API keys needed in the app** — Claude generates the data at skill-run time
- **Storage**: `window.storage` (Claude artifact storage) with `localStorage` fallback
- **Fonts**: Crimson Pro (French text) · DM Sans (UI) · Noto Sans SC (Chinese)
- **Per-article isolation**: vocab and SR progress stored independently per article slug

---

## 📋 Requirements

- A [Claude.ai](https://claude.ai) account (Pro recommended for longer articles)
- To run the output `.jsx` file locally: [Node.js](https://nodejs.org) + [Vite](https://vitejs.dev)

```bash
npm create vite@latest my-reading-app -- --template react
cd my-reading-app
# Replace src/App.jsx with the downloaded file
npm install && npm run dev
```

---

## 📁 Repository structure

```
french-reading/
├── README.md      ← this file
├── README_ZH.md   ← 中文说明
└── SKILL.md       ← the skill instructions for Claude
```

---

## 📄 License

MIT — free to use, modify, and share.

---

## 🙏 Credits

Developed iteratively with Claude (Anthropic) as a personal French learning tool.

