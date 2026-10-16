# Handoff: deploy the exam-week schedule to GitHub Pages

You are receiving a finished static website. Your job is to publish it on GitHub Pages for the user. Do not redesign or rewrite it.

## Files in this bundle

| File | Purpose |
|---|---|
| `index.html` | The complete site: one self-contained page (inline CSS and JS, Google Fonts via `<link>`). Ready to deploy as is. |
| `.nojekyll` | Empty file. Tells GitHub Pages to serve files as-is instead of running Jekyll. Keep it in the repo root. |
| `DEPLOY.md` | Human-readable deployment guide (stack, steps, troubleshooting). |
| `CLAUDE.md` | This file. Not needed on the site, but harmless to commit. |

## What the site is

A day-by-day study schedule (17 to 29 Oct 2026) for a medical student's exam week, written in Brazilian Portuguese. Each study block has a checkbox; ticks are saved in `localStorage` under the key `cronograma-provas-2026`. There is no backend, build step or dependency to install.

The schedule is editable in the page itself ("Editar" button): days, blocks, exams, subjects and notes. Edits are saved in `localStorage` under `cronograma-provas-dados`; the `DAYS` array is only the default schedule used when nothing is saved. "Criar com IA" shows a prompt for an LLM and imports the JSON it returns (schema in `aiPrompt()`); "Exportar/Importar backup" moves data between devices.

## Deployment task

1. Ask the user for their GitHub username and the repository name they want (suggest `cronograma-provas`). The repository must be **public** if they are on a free GitHub plan.
2. Create the repository with these files in the **root**: `index.html`, `.nojekyll` and, optionally, `DEPLOY.md` and `CLAUDE.md`.
3. Enable Pages: **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: `main`, folder `/ (root)` → Save**.
   With the GitHub CLI, the equivalent is:
   ```bash
   gh repo create <repo> --public --clone
   cd <repo>
   cp /path/to/bundle/{index.html,.nojekyll,DEPLOY.md,CLAUDE.md} .
   git add -A && git commit -m "Add exam week schedule" && git push -u origin main
   gh api -X POST repos/<user>/<repo>/pages -f "source[branch]=main" -f "source[path]=/"
   ```
4. Wait for the "pages build and deployment" run in the Actions tab to finish, then open `https://<user>.github.io/<repo>/` and check that the day cards render and a checkbox stays ticked after reload.
5. Give the user the live URL.

## Rules

- Do not change the content of `index.html` unless the user asks. If they ask to change the default schedule, change only the `DAYS` array inside the `<script>` block (visitors who already edited their schedule keep their saved version).
- Do not add a framework, bundler or GitHub Actions workflow; none is needed.
- Never ask the user to paste a GitHub password or token into chat. If authentication is required, have them run `gh auth login` themselves or do the steps in the GitHub web interface.
- The user prefers replies in Brazilian Portuguese.
