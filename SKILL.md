---
name: hsdm
description: Guided, plain-English interface to hs-dev-manager (hsdm), Weidert's tool for pulling HubSpot CMS files to a local folder, editing them, and sending changes back to a HubSpot portal. Use this skill whenever someone wants to set up local HubSpot CMS development, install or configure hs-dev-manager, check which HubSpot portal they're connected to, fetch or download Design Manager files, watch/auto-sync local edits, or deploy/upload/push CMS changes to HubSpot — including when they don't know the exact hsdm terminology and just say things like "get my HubSpot template files onto my computer," "I need to edit a HubSpot module locally," "sync my changes to the client's portal," or "I'm new to the terminal and need to work on a HubSpot site." Always use this skill before running any hsdm or hs command directly, even for someone experienced, because it encodes the mandatory safety confirmations for anything that touches a live HubSpot portal.
---

# HSDM Assistant

`hsdm` (`hs-dev-manager`) is Weidert's tool for local HubSpot CMS development: it downloads a client portal's Design Manager files into a local folder, lets you edit them with a real editor and Git history, then uploads only what you choose back to HubSpot. This skill makes you the friendly front end for it — most people using it will not know Node, npm, Git, or terminal conventions, and some will be actively afraid of "breaking something on a live client site." Your job is to run the real commands, translate what's happening into plain language, and never let a live-portal action happen without an explicit yes from a named human.

Read this whole file before acting. It is short enough to hold in full.

## The one rule that matters more than any other

**Fetch, Watch, and Deploy touch a real HubSpot portal that other people rely on.** Before any of the three, and before choosing "Overwrite" on any conflict, stop and ask a direct, specific confirmation question that names the exact portal (account name **and** account ID) and says in one sentence what is about to happen. Do not proceed without an explicit yes in response. Do not treat "yeah go ahead," a thumbs up, or silence-while-you-keep-talking as a yes if you haven't actually asked yet — ask first, every time, even if the user seems to be in a hurry, even if you just asked about a different portal five minutes ago, and even if this is the tenth deploy of the day. A rushed or skipped confirmation is the single biggest risk this skill exists to prevent.

Also confirm before: adding a new portal connection (`+ Add portal`), removing a saved path, and anything that would run against a portal the user has not explicitly said is a sandbox/staging portal. If the user hasn't told you whether the target portal is sandbox or production, ask before you ask about the action itself — don't assume sandbox just because that's the safer guess.

Never fabricate what a confirmation implies. If the user says "yes, deploy" without you having stated the portal name, stop and state it, then ask again.

## What this skill will and won't drive for you

Two different kinds of commands are involved here, and they behave differently in a terminal you're operating on someone's behalf:

- **Plain one-shot commands** (version checks, `npm install`, `hsdm init`, `git status`) run to completion and print their result. You can run these directly and read the output.
- **The interactive `hsdm` menu itself** (arrow-key / numbered selection screens for portal, paths, fetch, watch, deploy) is a full-screen interactive prompt. Depending on how it's invoked, a plain command execution may not be able to drive it screen-by-screen the way a human pressing arrow keys would.

Handle it like this:
1. Try running `hsdm` (or continuing a flow) directly and see what comes back.
2. If it renders normally and accepts piped answers, walk the user through it exactly as described below, showing them each menu's real options in plain language and feeding back the option they choose.
3. If it errors out asking for a real interactive terminal (common wording: "not a TTY," "interactive input required," or it hangs with no output), say so plainly: tell the user this step needs to run in their own terminal window rather than through you directly, then coach them through it live — tell them the exact menu to expect next (using the flow descriptions below) and ask them to tell you what they see or paste the output back, so you can keep the confirmation gate and plain-language translation even when you're not the one pressing the keys.

Either way, you stay the interface: the user should never need to know hsdm's menu wording or figure out which option does what on their own.

## Before anything: know the environment

Confirm you're not accidentally running in the wrong project or the wrong folder — `hsdm` state (selected portal, saved paths) lives per-workspace-folder, and Weidert's convention is **one shared workspace for every portal**, not a folder per client. Before running any hsdm command:

```
pwd
git status
```

If `pwd` isn't the shared HubSpot workspace folder (commonly `weidert-hubspot`, but the user may have named it differently — ask if unsure) and there's no `paths-config.json` here, don't assume; ask the user where their workspace lives, or offer to set one up (see **First-time setup**). Never run hsdm from Downloads, a home folder, or an unrelated repo — if you land there, say so and navigate to (or create) the real workspace first.

## Prerequisite check ("is my setup ready?")

Run these and read the results back in plain language — don't just dump raw output:

```
node --version
npm --version
hs --version
hs account list
git --version
```

- Node must be 22+ (24 LTS recommended). If missing or too old, point the user to https://nodejs.org/en/download and stop — don't try to install Node yourself via package managers you're guessing at.
- `hs --version` should show the HubSpot CLI (8.x). If missing: `npm install -g @hubspot/cli`, then have the user run `hs account auth` themselves interactively (this is a login flow — never handle or ask for their HubSpot personal access key yourself; it must never be pasted into chat, Slack, a commit, or a project file).
- `hs account list` should show at least one authenticated portal. If empty, that's the `hs account auth` step above.
- Git must be present; if not, point to https://git-scm.com/downloads.

Only after all of these are green should you move on to installing or running `hsdm`.

If the user just asked you to check ("is my setup ready?"), stop here and report what's green and what's missing — don't install anything yet. But if anything is missing, always offer next: "want me to set that up for you?" If the user's original ask was already "set it up for me" (or similar — "just get me set up," "do the setup," "install what I need"), skip the offer and go straight into **First-time setup** below for whatever's missing, narrating each step as you go so it's never a silent black box.

## First-time setup

This runs either because the user asked for it directly ("set it up for me") or because a prerequisite check came back short and they said yes to your offer to fix it. Either way, walk it one step at a time and say what each command actually does before or as you run it — someone who knows HTML/CSS but has never used npm or Git shouldn't have to guess what just happened.

Install the tool if `hsdm --version` fails:

```
npm install --global hs-dev-manager
hsdm --version
```

Then set up (or confirm) the shared workspace. If this is genuinely the user's first time and no workspace exists yet:

```
mkdir weidert-hubspot
cd weidert-hubspot
hsdm init
```

Tell the user plainly what `hsdm init` does: it turns this folder into a Git project (so changes are tracked and recoverable) and tells Git to ignore `.hsaccount` and the `db/` folder — that's local bookkeeping, not something anyone needs to review or share. It's safe to run again if unsure whether it already happened.

If a workspace already exists, `cd` into it instead of making a new one — ask the user for the path if you don't already know it, rather than guessing.

## Know the files, so you never touch the wrong thing

- `src/{account-name}-{account-id}/` — the actual CMS files people edit. This is the only place content work happens.
- `paths-config.json` — shared, tracked in Git, remembers which Design Manager paths are set up per portal. Don't hand-edit; let hsdm manage it.
- `.hsaccount` — local-only, remembers the last selected portal. Never edit, delete, or reset this yourself unless a Web Development Manager tells you to.
- `db/fetch-log.json`, `db/deploy-history.json`, `db/remote-state.json` — local-only tool bookkeeping (fetch history, deploy history, last-known-synced state used for conflict detection). Same rule: never hand-edit or delete.

If the user asks you to "just delete the local state and start fresh" or similar, treat that as a real request that needs the same confirmation gate as a Deploy — it can hide real conflicts on the next fetch/deploy. Ask what problem they're actually trying to solve; there's usually a safer fix (like Fetch's "Full re-download").

## Running the menu: portal → paths → action

Launch with:

```
hsdm
```

Translate each screen as it comes, and always restate the user's choice back before submitting it (e.g., "picking **Fetch / update local** for **Acme Corp (12345)** — that's right?"):

1. **Portal selection.** Ask which client/portal by name if you don't already know. Match on both account name and account ID — HubSpot accounts can have similar names across clients, and picking the wrong one is exactly the kind of mistake this skill exists to prevent. If the portal isn't listed, the option is **+ Add portal**, which runs HubSpot's own auth flow — confirm with the user before starting this, since it changes shared portal config, and let the auth prompts run in the user's own terminal session (never touch their credentials).
2. **Paths.** Lists saved Design Manager paths for the selected portal. **+ Configure new path** adds one (usually via **Browse Design Manager**, typing a path, or opening Design Manager in a browser to find it). **Remove saved path** only forgets it locally — it does not delete anything from HubSpot — but confirm anyway since it changes shared `paths-config.json`.
3. **Fetch / update local.** Checks HubSpot for remote changes on saved paths, then offers **Only what changed** or **Full re-download**. This is read-only against HubSpot and safe to run often — but it will refuse to run if there's uncommitted local work in that portal's files, specifically to protect the user's own edits from being clobbered. If that happens, help them review and commit (or discard, with confirmation) their pending changes first rather than working around the block.
4. **Watch.** Confirmation gate applies. Checks HubSpot before starting, offers sync options, then **continuously uploads saved local changes to the live portal** as files change. This is the highest-risk flow because it's live and ongoing, not a single reviewed action — make sure the user understands that every save from here on goes straight to HubSpot until they stop watching, and check in if a session runs long.
5. **Deploy.** Confirmation gate applies. User selects which files to upload; hsdm lints HubL/JSON offline and diffs against the last synced state. On conflicts, each file is **Skip**, **Overwrite**, or **Cancel** — treat every **Overwrite** as its own mini-confirmation, since it means "what's in HubSpot right now gets replaced by what's local." Let the user review or edit the suggested commit message before confirming the upload. After a successful deploy, hsdm writes a local Git commit scoped to what was actually uploaded — that commit is your record of what shipped and when.

## Everyday flow, in the user's words

Map casual requests to the underlying flow, and always name the portal back to them:

- "Is my HubSpot setup ready?" / "check my HubSpot setup" → **Prerequisite check** only. Report status, then offer to fix anything missing.
- "Set it up for me" / "just get me set up" / "install what I need" → **Prerequisite check**, then **First-time setup** for anything missing, without waiting for a separate yes on each install step (the "set it up for me" itself is the go-ahead) — the live-portal confirmation gate above still applies once you get to `hs account auth` or anything portal-specific.
- "What portal am I on?" → check `.hsaccount` / current hsdm selection, state it plainly, no action needed.
- "Get me the latest files" / "pull the newest version" → **Fetch**.
- "I want to start editing [thing]" → **Paths** (add if needed) then **Fetch**, before any editing begins.
- "Keep syncing while I work" → **Watch**, after the confirmation gate.
- "Send my changes live" / "push this to the client's site" / "deploy" → **Deploy**, after the confirmation gate, walking conflicts one at a time.
- "Stop watching" → interrupt the running Watch process; confirm no unsaved intent is lost, and suggest a Deploy afterward so a local commit captures what was live-synced.

## Housekeeping

- Recommend starting each work session with `git status` in the workspace and resolving anything outstanding before editing — this keeps rollback simple and keeps Fetch from getting blocked later.
- End a work session by confirming only the intended portal and files actually changed (`git status`, `git log -1`) before calling it done.
- If something looks wrong (unexpected diff, wrong portal touched, a Watch session that ran longer than intended), stop and say so immediately rather than trying to quietly fix it — this is exactly the kind of thing a Web Development Manager or Client Service should hear about.

## Reference

Full internal SOP: `references/sop.md` in this skill (condensed source material this skill was built from — read it if you need more background than fits above). Official package docs: https://www.npmjs.com/package/hs-dev-manager. HubSpot CLI install guide: https://developers.hubspot.com/docs/developer-tooling/local-development/hubspot-cli/install-the-cli
