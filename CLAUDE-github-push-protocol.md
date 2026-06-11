# CLAUDE.md — claude-skills (Streetwise Consultancy)

Project context for Claude when working in this repo. **Inherits from the master operating doc** at `~/Documents/streetwise-consultancy/CLAUDE-master-operating.md` — read that first for business context (pillars, service lines, voice rules). This file is the engineering layer only.

## Repo

- Remote: `https://github.com/AntonioC92/claude-skills.git`
- Default working branch: `Projects`
- Contents: skill markdown files (`sw-*.md`) used by the Streetwise stack (paid media, automation, content, sales). Some skills originated under the legacy Caruso Martech setup and were brought forward into Streetwise during the 2026 merger — naming convention is `sw-` going forward.

## Working preferences

**No permission prompts for subfolders.** Don't ask before reading, writing, or editing inside any subfolder of this directory. The whole tree is in scope. Just do the work — read, edit, create files anywhere under `claude-skills/` without checking first.

This applies to:
- Creating new skill files in this folder or any nested folder
- Editing existing `sw-*.md` files
- Running git commands inside the repo
- Reading any file under the project tree

Stop and ask only for: destructive actions (force-push, history rewrite, file deletion of more than one file at a time), changes to remote settings, or actions outside this directory tree.

## GitHub authentication

The GitHub Personal Access Token is stored in `.env` (gitignored, never committed).

When pushing or doing any authenticated git operation, source the env file first:

```bash
set -a; source .env; set +a
```

Then push using the token-embedded URL:

```bash
git push "https://${GITHUB_USERNAME}:${GITHUB_PAT}@github.com/AntonioC92/claude-skills.git" "$(git branch --show-current)"
```

Or for a one-liner push of the current branch:

```bash
set -a; source .env; set +a && \
  git push "https://${GITHUB_USERNAME}:${GITHUB_PAT}@github.com/AntonioC92/claude-skills.git" \
  "$(git branch --show-current)"
```

For pull / fetch, the same pattern works with `git pull` / `git fetch` substituted in.

**Never** echo `$GITHUB_PAT` in command output, never paste it into a commit message, never include it in a file other than `.env`, and never push the `.env` file. If `.env` shows up in `git status`, stop and verify `.gitignore` is working before continuing.

If a push fails with a 401/403, the token may be expired or rotated — tell Antonio and ask for a new one rather than guessing.

## Standard workflow when adding/editing a skill

1. Edit or create the relevant `sw-*.md` file(s).
2. `git add` the changed files (avoid `git add .` unless you've checked `git status` first — never accidentally stage `.env`).
3. Commit with a clear message describing the skill change.
4. Push using the pattern above.
5. Confirm the push succeeded and report the commit SHA + branch.

## Build conventions

This repo follows Streetwise build conventions (carried forward from the legacy Caruso Martech AI Marketing Intelligence Engine):
- Modular skills, one file per skill
- `sw-` prefix for Streetwise skills
- Skills are markdown with YAML frontmatter where applicable
- Reference the `sw-workflows-architecture-build-protocol.md` and `sw-paid-campaigns-data-normalization-schema.md` files when building anything that touches the n8n engine
- For new skills that are bundled (multiple files like SKILL.md + reference files), place them in a subfolder named after the skill (e.g. `sw-content-viral-hooks/SKILL.md` + supporting files). The subfolder name IS the skill identifier.
