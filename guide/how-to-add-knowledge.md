# How to add knowledge

Explains how the site is organized and how to add a new note. 

## How the site is structured

```
ai-learning-journey/
├── index.html          ← the app + config 
├── assets/custom.css   ← styling 
├── _sidebar.md         ← the navigation
├── notes.md            ← the welcome page (the site opens here)
├── _templates/
│   └── knowledge-template.md   ← copy this to start a new note
│
└── fastapi/            ← SECTION = one folder
    ├── README.md       ←   section overview
    └── day-01/         ← SUBSECTION = one folder per day
        ├── README.md   ←   day overview
        └── pydantic-annotated.md   ← NOTE = the actual content
```

Three levels: **section** (folder) → **day** (folder) → **note** (`.md` file).
A section can also be flat — just a folder with notes, no day level.

---

## ➕ Add a new note ( 3 steps ) 

### Step 1 — Create the folders + file

```
fastapi/README.md                     ← section overview (one line is fine)
fastapi/day-02/README.md              ← day overview
fastapi/day-02/dependency-injection.md  ← the actual note
```

Copy [`_templates/knowledge-template.md`](_templates/knowledge-template.md) as a
starting point for the note

### Step 2 — Link it in the sidebar

Open [`_sidebar.md`](_sidebar.md). **Indent 2 spaces per level:**

```markdown
- [FastAPI](fastapi/README.md)
  - [Day 01](fastapi/day-01/README.md)
    - [Pydantic & Annotated](fastapi/day-01/pydantic-annotated.md)
  - [Day 02](fastapi/day-02/README.md)
    - [Dependency Injection](fastapi/day-02/dependency-injection.md)
```

Paths are always written **from the site root**, even for nested notes.

Any item that has children gets a chevron and can be collapsed/expanded by
clicking it. The section containing the page you're on stays open automatically.

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


