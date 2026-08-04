# 🧭 How to add knowledge

This page explains how the site is organized and how to add a new note. The whole
process takes about a minute. (This guide is local-only — it's not linked
anywhere on the published site.)

## How the site is structured

```
ai-learning-journey/
├── index.html          ← the app + config (rarely touched)
├── assets/custom.css   ← styling (right sidebar, light/dark theme)
├── _sidebar.md         ← the RIGHT-hand navigation  ⭐ you edit this
├── _coverpage.md       ← the landing splash screen
├── README.md           ← the Home page
├── _templates/
│   └── knowledge-template.md   ← copy this to start a new note
│
└── <section-folder>/   ← one folder per SECTION (currently empty — add your own)
    └── README.md       ←   the section's overview / index
```

**The rule of thumb:** one **section** = one **folder**. One **note** = one
**`.md` file** inside that folder. The right sidebar (`_sidebar.md`) is the map
that ties them together.

---

## ➕ Add a new note (3 steps)

### Step 1 — Create the section (if it doesn't exist yet) and the note

Example: your first note is about tokenization, in a new "Foundations" section:

```
foundations/README.md              ← section overview (one line is fine)
foundations/tokenization.md        ← the actual note
```

Copy [`_templates/knowledge-template.md`](_templates/knowledge-template.md) as a
starting point for the note, or just write plain markdown. A good note has a
title, a source line, and a short summary at the top.

### Step 2 — Link it in the sidebar

Open [`_sidebar.md`](_sidebar.md) and add lines for the section and note:

```markdown
- [Foundations](foundations/README.md)
  - [Tokenization](foundations/tokenization.md)
```

### Step 3 — Publish

Commit and push:

```bash
git add .
git commit -m "Add note: tokenization"
git push
```

GitHub Pages rebuilds automatically. Your note is live at
`https://donghuynh0.github.io/ai-learning-journey/` within a minute or two.

---

## 👀 Preview locally before pushing (optional)

Because Docsify needs a web server (not `file://`), run this from the repo folder:

```bash
python3 -m http.server 3000
```

Then open <http://localhost:3000> in your browser. Edit files, refresh, and see
changes instantly. Stop the server with `Ctrl+C`.

---

## Tips for good notes

- **One idea per file.** Small, focused notes are easier to find and update.
- **Always add a source link** so you can trace where you learned it.
- Use the **status tag** (`draft` / `complete`) so you know what to revisit.
- Use blockquote callouts for emphasis:

```markdown
> [!TIP]
> This renders as a highlighted callout box.
```

- Use the **search box** (top of the sidebar) to find anything fast.
