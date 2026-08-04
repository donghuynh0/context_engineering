# How to add knowledge

Explains how the site is organized and how to add a new note. 

## How the site is structured

```
ai-learning-journey/
├── index.html          ← the app + config 
├── assets/custom.css   ← styling 
├── _sidebar.md         ← the navigation
├── _coverpage.md       ← the landing screen
├── README.md           ← the Home page
├── _templates/
│   └── knowledge-template.md   ← copy this to start a new note
│
└── <section-folder>/   ← one folder per SECTION 
    └── README.md       ←   the section's overview / index
```

One **section** = one **folder**. One **note** = one
**`.md` file** inside that folder. 

---

## ➕ Add a new note ( 3 steps ) 

### Step 1 — Create the section 

```
foundations/README.md              ← section overview (one line is fine)
foundations/tokenization.md        ← the actual note
```

Copy [`_templates/knowledge-template.md`](_templates/knowledge-template.md) as a
starting point for the note

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


Open <https://donghuynh0.github.io/ai-learning-journey/>

---

## Preview locally


```bash
python3 -m http.server 3000
```

Open <http://localhost:3000> 

---

## Tips for good notes

- **One idea per file.** Small, focused notes are easier to find and update.
- **Always add a source link** so you can trace where you learned it.
- Use the **status tag** (`draft` / `complete`) so you know what to revisit.
- Use blockquote callouts for emphasis:


