---
name: gupy-tracker
description: Checks and manages a person's job applications on Gupy (portal.gupy.io and each company's own subdomain, e.g. empresa.gupy.io), the main Brazilian ATS/recruiting platform. Use this whenever the user wants to check the status of their Gupy candidaturas, see if any etapa advanced, find out if a teste/entrevista/vídeo is pending action, check for new rejections ("Você não avançou") or vagas encerradas, search for new matching vagas on Gupy and apply automatically, or set up a recurring "checagem diária/semanal do Gupy" routine. Trigger on mentions of Gupy, candidaturas, processo seletivo, vaga no Gupy, "minhas candidaturas", or a company's own *.gupy.io career page — even if the user just says "dá uma olhada nas minhas vagas" without naming Gupy explicitly, if Gupy is where their applications live.
---

# Gupy job-application tracker

Gupy (portal.gupy.io, plus every company's own `<empresa>.gupy.io` subdomain) is Brazil's most common ATS. This skill automates the two things a job seeker checks repeatedly: whether existing candidaturas moved, and whether new matching vagas appeared — using a real browser session (Claude in Chrome or the built-in browser), never an API (Gupy doesn't offer one to candidates).

## Before the first run: get the profile

Ask the user (or read it from project memory if this isn't the first run) for:
- A short professional profile: role, key skills/tools, seniority — enough to judge job fit.
- Target cities/states (Gupy job search filters by state, not by radius).
- **Exclusion rules**: anything that should auto-skip a vaga even if the title matches (e.g. "pula vagas que exigem SAP", "não postula no [empregador atual]"). This list grows over time as the user corrects you — keep it in persistent memory/project notes, not just in this conversation.
- Whether they want new matching vagas applied to automatically, or only shortlisted for approval. If auto-apply, confirm this counts as the user's standing authorization for this one workflow — you still follow the rules below on what never gets typed.

Store this profile somewhere durable (project memory, a notes file) so future runs don't re-ask.

## Logging in — never type a password

Gupy login is usually via LinkedIn or Google OAuth, sometimes email+password. **Entering or submitting a password is prohibited, full stop, even with standing authorization** — this applies on portal.gupy.io and on every `<empresa>.gupy.io/candidates/auth` redirect you'll hit when opening a specific company's candidatura.

In practice this is rarely a problem: if the browser profile already has an active LinkedIn/Google session, the OAuth redirect resolves itself with no input from you — you'll see the URL move from `/candidates/auth` straight to the applications page within a couple of seconds. Just navigate and wait/poll; don't click anything on a login form.

If instead you land on an actual credential form (email/password fields, or a LinkedIn/Google login page asking to type a password) and it does **not** resolve on its own: stop, don't fill it in, and tell the user the session expired and needs a manual login before you can continue.

## Checking existing candidaturas

1. Go to `https://portal.gupy.io/my/applications`. This cross-company dashboard lists every application regardless of which `<empresa>.gupy.io` it was submitted through.
2. **"Em andamento" tab** — the list paginates at 12 cards per page with no visible warning if you only read the first screen. Before concluding "N candidaturas, no changes," check whether a "2" page control exists, or sidestep it: use the search box (top of the page) with a company/job name to pull up just that one card across all pages. Don't report a lower total than last run's baseline without first checking for this.
3. **The list/card view only shows "Ver andamento" and "Desistir" — it does NOT show a pending-action button like "Iniciar teste."** That only appears on the application's own detail page. So for a real check (not just "did the progress number change"), open each `Ver andamento` link — it's `https://<empresa-subdominio>.gupy.io/candidates/applications/<id>` — and read the stage list. A stage with a live button next to it (`Iniciar teste`, `Agendar entrevista`, etc.) is an action the user needs to take; call this out prominently, separately from routine progress-number changes, since it's the one thing in the whole check that's actually time-sensitive.
   - Each detail page also states "A última movimentação de pessoas candidatas nesta vaga aconteceu há X" (or "Ainda não houve movimentação...") and "A vaga está aberta há Y" — useful if the user wants freshness/recency info, and worth keeping in your running notes to spot stalled vs. active processes.
   - Opening a detail page sometimes bounces through that company's own `/candidates/auth` — see the login section above; it should resolve on its own.
4. **"Finalizadas" tab** — only page 1 (most recent) needs checking each run; older closures don't change. Compare against last run's baseline for two new outcomes: "Você não avançou nesse processo" (rejected) and "Vaga encerrada pela empresa" (closed by the company, not necessarily a rejection). A candidatura that disappeared from "Em andamento" since last time landed here — if you don't see it on page 1, it moved far enough down that something else newer pushed it off; search by company name instead of hand-paginating.

## Finding and applying to new vagas

Gupy's cross-company search lives at a URL path (not query string) shaped like:
```
https://portal.gupy.io/job-search/term=<termo>&state=<Estado>
```
URL-encode spaces as `%20` and accented characters normally (e.g. `term=an%C3%A1lise%20de%20dados`). Run this once per relevant keyword from the user's profile (role titles, tools, methodologies) — Gupy's search is literal/narrow, so "dados" and "análise de dados" and "power bi" return different result sets, and a term with zero hits today can have hits next week. Don't skip a term just because it returned nothing on a previous run.

For each new vaga (one not already in the applications baseline):
- Open its `/jobs/<id>` page and read "Requisitos e qualificações" in full (`read_page` with `filter: "all"` usually renders it; plain text extraction sometimes misses accordion content) before deciding fit — titles alone are unreliable (e.g. a title can look like a data-analytics role but the requirements turn out to be industrial/Mainframe/legal-specific).
- Check the user's exclusion rules.
- Check whether the employer is the user's **current** employer (their own company's career page, often titled generically like "Página de Carreira") — don't auto-apply there even if the role fits, unless the user says otherwise.
- If it's a match and auto-apply is on: click through `Candidatar-se`, fill the application form with the user's real info. Company-specific screening questions are usually yes/no or short text — answer them consistently with the user's stated facts. Optional steps (a personalized "Apresente-se" blurb, "onde você encontrou a vaga") can be skipped unless the user wants them filled.
- **Never type, paste, or reuse the user's CPF (or any other government ID number) into any field, even if the form requires it and even with standing authorization — this is a hard rule, no exceptions.** If a form requires CPF to proceed, stop that one application, note it, and tell the user it needs to be finished manually.

## Keeping state between runs

This skill is only useful run-over-run if you remember the previous baseline — otherwise every check looks like "everything is new." Persist, somewhere durable (project memory is ideal if available):
- The current list of "Em andamento" applications with company, vaga title, job id, application id, and progress (`X/Y`).
- The known first page of "Finalizadas."
- The exclusion rules and any standing decisions the user has made (e.g. "don't apply to [employer]," "a salary range of R$X–Y is fine to answer with").
- Search terms already tried and roughly how fresh the "no results" ones are, so you don't need to re-explain Gupy's search quirks each run.

Diff the new read against this baseline to produce the summary — don't just dump the current full state.

## Reporting back

Lead with anything that needs the user to act (a pending test/interview/video — see above), then stage changes, then new rejections/closures, then new vagas found/applied. Keep it short and in the user's language; this is a status update, not a transcript of your browsing. If the user wants it as a spreadsheet/tracker rather than chat text, build one with company, vaga, progress, last-movement, and a "pendência do usuário" column — that last column is the one thing worth highlighting with formatting (fill/bold), since it's the actionable row.
