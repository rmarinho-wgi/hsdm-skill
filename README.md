# hsdm — HubSpot Dev Manager Assistant (Claude Skill)

A Claude Code skill that gives non-terminal-expert Weidert staff a plain-English, conversational front end for `hs-dev-manager` (`hsdm`) — the tool used to pull HubSpot CMS files locally, edit them, and push changes back to a client's portal.

Instead of learning `hsdm`'s menu structure or terminal conventions, you tell Claude what you want ("get me the latest files for Acme Corp," "send my changes live") and it runs the real commands, explains each step, and — critically — **always asks for an explicit yes before anything touches a live HubSpot portal** (Deploy, Watch, or an Overwrite on conflict).

## What's in this repo

```
SKILL.md              Skill definition Claude loads: triggers, workflow, safety rules
references/sop.md      Condensed source SOP the skill was built from
tutorial/index.html     Standalone visual walkthrough (open in a browser)
```

**Live tutorial:** https://claude.ai/artifact/U9YwQJYMbLFniGWfR9WndF — this is the link to paste into the internal tool's "Link to output" field. `tutorial/index.html` in this repo is the same page, kept here so it isn't only reachable through claude.ai.

## Installing this skill

**Claude Code (CLI / desktop / web):**

1. Clone or add this repo, or copy its contents into a folder Claude Code loads skills from (for individual use: `~/.claude/skills/hsdm/`; for a project: `<project>/.claude/skills/hsdm/`).
2. Make sure `SKILL.md` (and the `references/` folder) end up in that folder.
3. Restart or start a new Claude Code session — the skill appears automatically once its `SKILL.md` is in a loaded skills directory, no further setup needed.

**Organization-wide:** see the "Shipping this skill" section below — this is meant to be registered through Weidert's internal skill catalog ("Ship a Skill" form) so every relevant team member gets it without manually copying files.

## Shipping this skill (internal "Ship a Skill" form)

Suggested values for the internal form:

| Field | Suggested value |
|---|---|
| Title | HSDM Assistant (HubSpot Dev Manager Helper) |
| Description | Plain-English, guided interface to `hs-dev-manager` for setting up local HubSpot CMS development, fetching Design Manager files, and safely deploying changes — built for people who don't want to learn terminal commands or hsdm's menus, with mandatory confirmation before anything touches a live portal. |
| Category | Client work (or Dev/Internal tooling, if that category exists) |
| Link to output | This repo's GitHub URL, and/or the `tutorial.html` walkthrough link |
| Time saved | Estimate — ask whoever onboards a non-technical teammate with this vs. without it |
| Fun fact | It will refuse to deploy or watch-sync to a live portal without you typing an explicit yes to a question that names the exact client and account ID — no silent live pushes, ever |

See `tutorial/index.html` (or the live link above) for a shareable, non-technical walkthrough.

## Safety model (why this exists)

`hsdm` itself already protects against a lot (blocked fetches on uncommitted work, offline lint, conflict diffs) — this skill adds the human layer on top: it never lets Fetch/Watch/Deploy or a conflict Overwrite happen without first stating the exact portal and asking for a real yes. See `SKILL.md` for the full rule set.
