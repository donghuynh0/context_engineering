# uv — commands I actually use

<p class="note-meta">
  <span class="tag">Completed</span>
  <span>Updated 2026-08-06</span>
  <a href="https://docs.astral.sh/uv/">Source: uv docs</a>
</p>

> A Rust-based Python package manager. Replaces `pip` + `venv` + `pipx` + `pyenv`, and is much faster.

---

## Install

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh   # macOS / Linux
uv --version
```

---

## Start a FastAPI project

```bash
uv init myapi          # creates pyproject.toml + .python-version
cd myapi
uv add fastapi --extra standard
uv run fastapi dev     # uv creates .venv and installs on first run
```

No `activate` needed — `uv run` executes inside the project env.

---

## Daily commands

| Command | What it does |
|---|---|
| `uv add <pkg>` | add a dependency (updates `pyproject.toml` + lockfile) |
| `uv add --dev pytest` | add a dev-only dependency |
| `uv remove <pkg>` | remove a dependency |
| `uv sync` | install exactly what the lockfile says |
| `uv lock` | re-resolve and update `uv.lock` |
| `uv run <cmd>` | run a command inside the project env |
| `uv tree` | show the dependency tree |

`uv add` / `uv remove` already re-lock and sync — you rarely call `uv lock` by hand.

---

## Python versions

```bash
uv python install 3.12    # download a Python
uv python list            # what's available / installed
uv python pin 3.12        # write .python-version for this project
```

---

## Run a tool without installing it

```bash
uvx ruff check .          # one-off, like npx
uv tool install ruff      # keep it on PATH
```

---

## Existing project using requirements.txt

uv has a drop-in `pip` interface:

```bash
uv venv                              # create .venv
uv pip install -r requirements.txt
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt         # make env match the file exactly
```

---

## Worth remembering

- **Commit `uv.lock`** — that's what makes builds reproducible.
- `uv sync` **removes** packages that aren't in the lockfile, so the env stays clean.
- `uv pip install` is *not* the same as `uv add` — it installs into the venv but does **not** record the dependency in `pyproject.toml`.

---

## Read more

- [uv docs](https://docs.astral.sh/uv/)
- [uv — Working on projects](https://docs.astral.sh/uv/guides/projects/)
- [FastAPI — Deployment / environments](https://fastapi.tiangolo.com/virtual-environments/)
