# CLAUDE.md

> **Purpose:** Persistent instructions for Claude when working in this project.  
> **Rule:** Edit specific sections. Never rewrite this file entirely.

---

## Project Overview

<!-- Update this section when the project scope changes -->

[Project name and one-line description]

---

## Research / Discovery

Before building anything, do a quick check:

- **Does something already exist that does this?** If so, use it or adapt its code rather than building from scratch
- **Do a couple of things share elements with what we need?** Consider combining or adapting code from multiple sources
- **Keep it brief** — don't spend excessive tokens on exhaustive research, just enough to avoid reinventing the wheel
- Document what was found (and what was ruled out) so the decision is clear

---

## Tech Stack

- **Language:** Python 3.x (default — but suggest alternatives if genuinely better suited to the task)
- **Web framework:** [Flask/FastAPI/None—update as needed]
- **Testing:** pytest (or equivalent for the chosen language)
- **Package manager:** pip (use requirements.txt) — adapt if using a different language/ecosystem

---

## Git Workflow

- **Always work on a branch** — never commit directly to main
- **At the start of each session, ask the user:** "Would you like to continue on an existing branch or start a new one?"
  - If continuing: check out the existing branch and pull latest changes
  - If starting fresh: create a new branch from main (e.g. `git checkout -b feature/description`)
- When work is ready, push the branch and create a PR to merge into main
- Do not push directly to main

## Development Workflow

```bash
# 1. Continue on an existing branch OR create a new one (ask the user)
git checkout existing-branch  # continuing
# or
git checkout -b feature/short-description  # starting fresh

# 2. Create virtual environment (first time only)
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run tests
pytest

# 5. Run local server (if web-based)
python app.py  # or: flask run / uvicorn main:app

# 6. Before committing
pytest && git add -A && git commit -m "message"
```

---

## Working Principles

### Plan Before Building
- Enter plan mode for any non-trivial task (3+ steps or architectural decisions)
- Write detailed specs upfront to reduce ambiguity
- If something goes sideways, **stop and re-plan immediately** — don't keep pushing
- Use plan mode for verification steps, not just building

### Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### Autonomous Bug Fixing
- When given a bug report: just fix it — don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

### Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### Self-Improvement Loop
- After any correction from the user: update `tasks/lessons.md` with what went wrong and a rule to prevent it
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for the relevant project

---

## Code Standards

- Use type hints for function signatures
- Docstrings for public functions
- Keep functions under 30 lines where possible
- Prefer explicit over clever

---

## File Structure Conventions

```
project/
├── src/              # Main source code
├── tests/            # Test files (mirror src/ structure)
├── requirements.txt  # Dependencies
├── app.py            # Entry point (if web-based)
└── README.md         # User-facing documentation
```

---

## Known Issues & Workarounds

<!-- Append new issues here as they're discovered -->

- None yet

---

## README Maintenance

- **Always update README.md before the end of a session** with meaningful changes
- The README should be clear and semi-technical — written for someone who wants to use the app, not deep-dive the internals
- Required sections:
  - **What it does** — a plain-English summary of the app's purpose
  - **How it works** — a brief, high-level explanation of the approach (no implementation minutiae)
  - **Installation** — step-by-step setup instructions
  - **Usage** — how to actually run and use the app
- Keep it concise and up to date — if the app changes, the README should reflect that

---

## Do NOT Do

<!-- Append items here when Claude makes mistakes -->

- Do not rewrite CLAUDE.md or LEARNINGS.md entirely—edit sections only
- Do not delete test files without explicit approval
- Do not change the directory structure without discussing first
- Do not commit or push directly to main — always use a branch
