# Handoff: deploy the year-round study schedule to GitHub Pages

You are receiving a finished static website. Your job is to publish it on GitHub Pages for the user. Do not redesign or rewrite it.

## Files in this bundle

| File | Purpose |
|---|---|
| `index.html` | The complete site: one self-contained page (inline CSS and JS, Google Fonts via `<link>`). Ready to deploy as is. |
| `.nojekyll` | Empty file. Tells GitHub Pages to serve files as-is instead of running Jekyll. Keep it in the repo root. |
| `DEPLOY.md` | Human-readable deployment guide (stack, steps, troubleshooting). |
| `CLAUDE.md` | This file. Not needed on the site, but harmless to commit. |

## What the site is

A year-round study planner for a medical student, in Brazilian Portuguese. One static page, no backend, build step or dependency.

- **Accounts (local).** Login with username + password. Each account's data is encrypted in `localStorage` (PBKDF2-SHA-256, 600k iterations → AES-GCM 256). Registry: `cronograma:contas`; data: `cronograma:dados:<id>`; session key: `cronograma:sessao` (sessionStorage, plus localStorage when "Manter conectado" is ticked). There is no password recovery and no cross-device sync; the "Exportar/Importar backup" menu moves data between devices. Requires a secure context (https or localhost).
- **Data model.** `periods` (date range + weekly routine: blocks per weekday) cover the year; `days` (map by date) override the routine on specific dates (e.g. the exam week); `exams` are separate events with countdown; progress is `done["AAAA-MM-DD|blockId"]`. A period that starts later wins when periods overlap.
- **Views.** Criar com IA (see below), Hoje (default: current day, ‹ › navigation, week strip, "Agora" card, next exams, last 7 days), Semana, Mês (calendar with progress), Revisões (spaced reviews D1→D7→D15→D30→D60, `STEPS` in the script), Planejar (routine editor, exams, subjects, custom days, notes, AI/backup). Routes are hash-based (`#/dia/2026-10-21`, `#/ia`, `#/semana`, `#/mes/2026-10`, `#/revisoes`, `#/planejar`). ← → keys and horizontal swipe move between days/weeks/months.
- `?hoje=AAAA-MM-DD&hora=HH:MM` simulates a date and time for checking the layout.
- `DAYS` (exam week, Oct 2026) and `ROUTINE` (weekly routine) are only the **sample** used when a new account picks "Cronograma de exemplo". Data saved by the old single-user version (`cronograma-provas-2026`, `cronograma-provas-dados`, `cronograma-provas-revisoes`) can be imported when creating an account ("Meu cronograma salvo neste navegador").
- **Criar com IA** is a central tab (`#/ia`, task pages `#/ia/<task>`). `AI_TASKS` in the script defines one guided task per job (tudo, rotina, materias, provas, reta, ajustar, atraso, revisoes, dicas, colar), each with its own prompt (`aiPrompt(id)`: interview questions → build → self-check → JSON schema limited to the task's keys), optional copy of the current plan (`aiContext`), live preview/validation of the pasted reply (`aiCheck`, accepts several ```json parts), replace/merge rules (`aiApply`; `days` are always per-date, `days_reset` returns dates to the routine, "ajustar" keeps ids so progress survives), a "correction message" to send back to the AI on errors, and an in-memory undo of the last import. JSON keys: subjects, periods, days, days_reset, exams, reviews, notes.
- New accounts can start with "Criar o meu com ajuda de uma IA", which opens `#/ia/tudo` on a blank plan.

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
4. Wait for the "pages build and deployment" run in the Actions tab to finish, then open `https://<user>.github.io/<repo>/` and create an account, check that today's card renders and a checkbox stays ticked after reload.
5. Give the user the live URL.

## Rules

- Do not change the content of `index.html` unless the user asks. If they ask to change the sample schedule, change only `DAYS` / `ROUTINE` inside the `<script>` block (existing accounts keep their saved data).
- Do not add a framework, bundler or GitHub Actions workflow; none is needed.
- Never ask the user to paste a GitHub password or token into chat. If authentication is required, have them run `gh auth login` themselves or do the steps in the GitHub web interface.
- The user prefers replies in Brazilian Portuguese.
