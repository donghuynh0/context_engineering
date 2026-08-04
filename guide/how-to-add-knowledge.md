# 🧭 How to add knowledge

This page explains how the site is organized and how to add a new note. The whole
process takes about a minute.

## How the site is structured

```
ai-learning-journey/
├── index.html          ← the app + config (rarely touched)
├── assets/custom.css   ← styling (sidebar on the right, colors)
├── _sidebar.md         ← the RIGHT-hand navigation  ⭐ you edit this
├── _navbar.md          ← the small top navigation bar
├── _coverpage.md       ← the landing splash screen
├── README.md           ← the Home page
├── _templates/
│   └── knowledge-template.md   ← copy this to start a new note
│
├── foundations/        ← one folder per SECTION
│   └── README.md       ←   each section's overview / index
├── prompt-engineering/
├── context-engineering/
├── agents/
│   ├── README.md
│   └── workflows-vs-agents.md  ← a note lives inside its section folder
├── rag/
└── resources/
```

**The rule of thumb:** one **section** = one **folder**. One **note** = one
**`.md` file** inside that folder. The right sidebar (`_sidebar.md`) is the map
that ties them together.

---

## ➕ Add a new note (3 steps)

### Step 1 — Create the markdown file

Put it in the right section folder. For example, a note about few-shot prompting:

```
prompt-engineering/few-shot-prompting.md
```

Copy [`_templates/knowledge-template.md`](_templates/knowledge-template.md) as a
starting point, or just write plain markdown. A good note has a title, a source
line, and a short summary at the top.

### Step 2 — Link it in the sidebar

Open [`_sidebar.md`](_sidebar.md) and add one line under the right section:

```markdown
- **Prompt Engineering**
  - [Overview](prompt-engineering/README.md)
  - [Few-shot prompting](prompt-engineering/few-shot-prompting.md)   ← new line
```

Optionally, also list it under **Notes in this section** in that section's
`README.md`.

### Step 3 — Publish

Commit and push:

```bash
git add .
git commit -m "Add note: few-shot prompting"
git push
```

GitHub Pages rebuilds automatically. Your note is live at
`https://donghuynh0.github.io/ai-learning-journey/` within a minute or two.

---

## 🆕 Add a whole new section

1. Create a folder, e.g. `evaluation/`.
2. Add an overview file `evaluation/README.md` (copy the shape of an existing one).
3. Add a new group to [`_sidebar.md`](_sidebar.md):

```markdown
- **Evaluation**
  - [Overview](evaluation/README.md)
```

That's it — the new section appears in the right sidebar.

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
