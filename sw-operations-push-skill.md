---
description: Push an updated skill file from the local streetwise-consultancy/claude-skills folder to its remote git repo
---

Push a skill file from the local `streetwise-consultancy/claude-skills/` folder to the connected remote git repository.

Replace `SKILL-FILENAME.md` with the actual filename (e.g. `content-antonio-tone.md`).

If your local skills folder is somewhere other than `~/Documents/streetwise-consultancy/claude-skills/`, update the path below before running.

// turbo
1. Stage, commit, and push to remote:
```
git -C /Users/Antonio/Documents/streetwise-consultancy/claude-skills add SKILL-FILENAME.md && git -C /Users/Antonio/Documents/streetwise-consultancy/claude-skills commit -m "Update SKILL-FILENAME skill" && git -C /Users/Antonio/Documents/streetwise-consultancy/claude-skills push
```
