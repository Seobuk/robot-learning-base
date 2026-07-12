---
name: setup
description: >
  Bootstrap an existing codebase that was just dropped, cloned, or pasted into the project:
  install its dependencies, analyze the actual code, then produce three things — a
  dev-environment guide, a persistent memory entry, and a CLAUDE.md tailored to the real code.
  Use this whenever the user brings in existing code and wants to set it up, onboard it, get it
  running, or "make a CLAUDE.md for this code" — e.g. "이 코드 세팅해줘", "기존 코드 분석해서
  CLAUDE.md 만들어줘", "개발환경 정리해줘", "onboard this repo", "bootstrap this project",
  "set up this codebase" — even if they never say the word "setup". Prefer this over a plain
  /init whenever there is existing code that must be installed and understood first.
license: MIT
---

# setup

Onboard an existing codebase: get it installed, understand it from the actual source, and leave
behind three durable artifacts so neither the user nor a future session has to re-derive how the
project works — a **dev-environment guide**, a **memory entry**, and a **CLAUDE.md** that matches
the real code.

## The one rule: ask before you change anything

This skill mutates the machine and writes files, so treat every change as something the user must
approve first. Before you run an install command, create a file, or overwrite an existing one,
**say exactly what you're about to do and wait for a yes.** Batch related actions into one clear
question rather than a stream of tiny prompts, but never install, write, or overwrite silently.
If a file (especially `CLAUDE.md`) already exists, show a merge/diff plan and get approval before
touching it — clobbering someone's existing config is the worst failure mode here.

Why: onboarding runs against code you don't own yet. A wrong install command or an overwritten
config is expensive to undo, and the user knows things about their setup that the code doesn't
show. Asking first is faster than guessing wrong.

## Workflow

Work through these phases in order. State a short plan up front and check off each phase as you go.

### 1. Detect & install (ask before running)

Figure out what the project is from its manifests, then propose the install commands and confirm
before running them. Match the ecosystem:

| Signal | Likely install (confirm first) |
|---|---|
| `package.json` (+ lockfile) | `npm ci` / `pnpm i` / `yarn` — match the lockfile |
| `pyproject.toml` / `uv.lock` | `uv sync` (or `pip install -e .`) |
| `requirements.txt` | `pip install -r requirements.txt` (in a venv) |
| `pixi.toml` | `pixi install` |
| `environment.yml` | `conda env create -f environment.yml` |
| `Cargo.toml` | `cargo build` |
| `go.mod` | `go mod download` |
| `pom.xml` / `build.gradle` | `mvn install` / `gradle build` |
| `Gemfile` | `bundle install` |
| ROS `package.xml` + `src/` | `colcon build --symlink-install` |
| `Dockerfile` / `compose.yaml` | offer the container path instead of host install |
| bare `Makefile` | read it — `make setup` / `make install` often exists |

Prefer the path the repo already documents (its own README/CONTRIBUTING) over your default. Pin to
the versions the lockfiles/configs imply — don't silently bump them. **Record the exact commands
that actually worked**; those verified commands are the ground truth for the guide and CLAUDE.md,
not what you assume should work.

Then verify the install landed: run the build, a quick test, or an import/smoke check. If it fails,
surface the error and the likely fix rather than papering over it.

### 2. Analyze the real code (read, don't assume)

Understand the project from the source before you write a word of documentation. Cover:

- **How to run / test / build / lint** — the actual entry points and scripts (`scripts` in
  package.json, `[project.scripts]`, `Makefile` targets, launch files, `main`/`__main__`).
- **Structure** — top-level layout, the few modules that matter, where config lives.
- **External surface** — env vars and secrets (`.env`, `.env.example`, config files), services/DBs,
  ports, external APIs. Note what must be provided but isn't committed.
- **Conventions** — formatter/linter config, test framework, language version, style already in use.
- **Generated / off-limits dirs** — build output, caches, vendored deps that shouldn't be edited.

For a large or unfamiliar repo, delegate this survey to a subagent (or the Explore agent) and keep
only the findings — you need the map, not every file in context. Ground every later claim in
something you actually read; that's the whole point of doing this before generating CLAUDE.md.

### 3. Generate the three artifacts (preview + confirm each)

Draft each artifact, show it (or its outline) to the user, and confirm before writing. Keep them
specific and testable — real commands and paths, not aspirations.

**a. Dev-environment guide** — propose a path (`docs/DEV_ENVIRONMENT.md`, or `SETUP.md` if simpler)
and confirm. Contents:

```markdown
# Dev Environment

## Prerequisites        # language + version, system deps, GPU/driver if relevant
## Install              # the exact, verified commands from phase 1
## Environment          # env vars / .env keys, where secrets come from
## Run / Test / Build   # verified commands, ports, expected output
## Gotchas              # anything that bit you during install, platform notes
```

**b. Memory entry** — write a `project`-type memory into the persistent memory directory named in
your system prompt's memory instructions (one file per fact + a one-line pointer in `MEMORY.md`).
Capture only what a future session couldn't re-derive from the code in seconds: the stack, how to
run it, non-obvious constraints, where things live. Convert relative dates to absolute. Follow the
exact frontmatter format the memory instructions specify. Confirm the slug/summary before writing.

Why: the memory persists across sessions, so the next time work touches this repo the "how do I run
this" cost is already paid.

**c. CLAUDE.md** — repo root. If one exists, **merge surgically** and show the plan first; do not
overwrite. Contents, tuned to what you found:

```markdown
# CLAUDE.md
<one-line: what this project is>

## Commands          # verified build/test/run/lint — the exact strings, agents guess these wrong
## Architecture      # 3-6 lines: the modules that matter and how they connect
## Conventions       # language version, formatter/linter, test framework, style to match
## Do not touch      # generated/vendored dirs; add deps to the manifest, not ad-hoc installs
## Secrets / safety  # read keys from .env (never commit); any domain-specific safety rules
```

Write in plain, direct language and explain the why behind rules — avoid all-caps MUST/ALWAYS,
which makes capable models over-trigger. State rules as "Use X when…", specific and checkable. If
`karpathy-guidelines` or a similar behavioral skill is installed, note that its principles apply
rather than re-inlining them.

### 4. Verify & report

Re-run the commands you documented to confirm they work as written (a guide with a wrong command is
worse than none). Then report: what was installed, what verified, and the three files created —
with their paths.

## When NOT to use this

- **Greenfield / empty repo** with no existing code to install — a plain `/init` fits better; there's
  nothing to analyze yet.
- **A one-off question** ("what does this function do?") — just answer it; don't run the whole
  onboarding.
- **Adding a feature to a repo that's already set up and documented** — use the existing CLAUDE.md;
  only re-run setup if the environment or stack materially changed.
