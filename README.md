# gupy-tracker

A [Claude](https://claude.com) skill that checks and manages job applications on [Gupy](https://gupy.io), the most common ATS/recruiting platform in Brazil — without ever typing a password or a CPF.

## The problem

If you're job-hunting through Gupy, you end up doing the same manual chore every day: open the dashboard, click into each application to see if a stage advanced, check whether a test or interview is sitting there waiting for you (spoiler: that button only shows up on the detail page, never in the list), check the "Finalizadas" tab for new rejections, and run a handful of keyword searches to see if anything new matching your profile popped up.

It's repetitive, it's easy to miss things, and the UI doesn't help: Gupy's "Em andamento" list silently paginates at 12 cards with no warning, and the only place a pending action (e.g. "Iniciar teste") actually appears is on each application's own detail page — never on the card in the list.

## The flow

This skill drives a real browser session (no API — Gupy doesn't expose one to candidates) through:

1. **Login** — relies entirely on an already-active LinkedIn/Google OAuth session. If a real credential form shows up and doesn't resolve itself, the skill stops and tells the user to log in manually. It never types or submits a password.
2. **Checking existing applications** — opens the cross-company dashboard, handles the silent pagination, and opens each application's own detail page (not just the list) to catch pending actions that the list view hides.
3. **Diffing against the last run** — compares against a saved baseline (company, vaga, progress, stage) so the summary only reports what actually changed.
4. **Finding new matching vagas** — runs the user's profile keywords against Gupy's job search, reads the full requirements (not just the title, which is often misleading), and checks the user's exclusion rules and current employer before applying.
5. **Reporting** — leads with anything that needs the user to act (a pending test/interview), then stage changes, then new rejections/closures, then new vagas found/applied.

## Rules that never break

- **Never types or submits a password**, on portal.gupy.io or any `empresa.gupy.io` subdomain, even with standing authorization to auto-apply.
- **Never types or reuses a CPF** (or any other government ID) in any field, for any reason. If a form requires one to proceed, that application is left for the user to finish manually.
- **Never auto-applies to the user's own current employer's career page**, even if the role matches.
- Exclusion rules the user sets (e.g. "skip anything requiring SAP", "don't apply to X") are persisted and checked before every application, not just remembered for one session.

## How to use it

Drop `SKILL.md` into your Claude skills folder (or add it as a skill in a Claude product that supports skills). Then just talk to Claude about your Gupy applications — "check my Gupy applications," "did anything move on my candidaturas," "look for new vagas that match my profile" all trigger it. The skill will ask for your profile (role, key skills, target cities/states, exclusion rules) the first time, and from then on keeps state between runs so each check only reports what's new.

## License

MIT — see [LICENSE](LICENSE).
