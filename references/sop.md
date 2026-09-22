# Reference: SOP — Developing with the CLI (hs-dev-manager)

Condensed from the internal SOP "SOP: Developing with the CLI" (Aug 07, 2026). This is background reading for the `hsdm` skill — the skill's SKILL.md is the operating instructions; this file is the source material behind them, kept for when more context is needed than fits in SKILL.md.

## Purpose

Set up a safe local workflow: choose the correct HubSpot portal, fetch files into a tracked Git repository, make and review changes locally, then upload only to the intended portal.

## Before you start

Check permission to access the HubSpot portal and to edit its Design Manager assets. Confirm you're working on a sandbox or staging portal. If unsure which portal to use, or permissions look wrong, contact Client Service or the Web Development Manager before continuing.

## Required access and tools

- Node.js 22+ (24 LTS recommended for a new setup)
- HubSpot CLI 8.x (official `hs` commands and authentication)
- Git (the Manager uses it to detect local changes and create scoped sync/deploy commits)
- A code editor with an integrated terminal (Cursor is the usual choice)

## Install the dependencies

Node.js, HubSpot CLI, and account authentication are covered by HubSpot's official guide — complete that first: https://developers.hubspot.com/docs/developer-tooling/local-development/hubspot-cli/install-the-cli

1. **Set up the HubSpot CLI and authenticate.** Install a supported Node.js version, install `@hubspot/cli` globally, run `hs account auth` and follow the prompts. Verify with `node --version`, `npm --version`, `hs --version`, `hs account list`. Never paste a HubSpot personal access key into Slack, Git commits, or project files.
2. **Install and identify Git.** `git --version`; set identity with `git config --global user.name "Your Name"` and `git config --global user.email "you@weidert.com"`.
3. **Install the HubSpot Manager Tool.** `npm install --global hs-dev-manager`, then `hsdm --version`. `hsdm` and `hs-dev-manager` are equivalent commands — use the shorter `hsdm` day to day. Package details: https://www.npmjs.com/package/hs-dev-manager

## Get situated locally

The recommended Weidert setup uses **one shared Git workspace for all managed HubSpot portals**, not a separate folder per portal. Run `hsdm` from that workspace root — it keeps every portal isolated under `src/{account-name}-{account-id}/` and stores portal-specific paths and sync state alongside it. Confirm your location with `pwd` and `git status` before running `hsdm`.

**Prepare a new project folder:**

```
mkdir weidert-hubspot
cd weidert-hubspot
hsdm init
```

`hsdm init` initializes Git when needed and adds `.hsaccount` and `db/` to `.gitignore` without changing existing rules. Safe to run again. If skipped, the Manager offers the same setup when a path is first configured.

Do not run the Manager from Downloads, a home folder, or another unrelated repository.

## Know the local files

**Work here:**
- `src/{account-name}-{account-id}/` — the HubSpot CMS files you edit.

**Tool-managed files (do not edit manually):**
- `paths-config.json` — remembers remote paths by portal; shared project configuration, tracked in Git.
- `.hsaccount` — remembers the last selected portal ID; local to your machine.
- `db/fetch-log.json` — records successful fetches; local to your machine.
- `db/deploy-history.json` — recent deploy records; local to your machine.
- `db/remote-state.json` — last known remote sync state, used for conflict checks; local to your machine.

Edit CMS code only under `src/{account-name}-{account-id}/`. Keep `paths-config.json` tracked in Git. `.hsaccount` and `db/` are local state ignored by `hsdm init` — don't change, delete, or reset them manually unless the Web Development Manager instructs you to.

## First run and initial fetch

Run `hsdm`.

1. Select a portal by checking both account name and account ID. If not listed, choose **+ Add portal** and complete the HubSpot CLI authentication flow.
2. After selecting the portal, choose **Paths**.
3. The Paths menu lists saved paths for the selected portal. **+ Configure new path** adds one; **Remove saved path** removes it from the Manager only (does not delete files from HubSpot).
4. Choose **Browse Design Manager**, type a path manually, or open Design Manager in the browser. Select the remote path needed. The Manager downloads it into `src/{account-name}-{account-id}/` and records the initial local sync commit. Development happens locally, not directly in the Design Manager editor.
5. On later sessions, choose **Fetch / update local** before starting a feature. The Manager checks HubSpot for remote changes and shows whether each saved path is current. If a path changed, choose **Only what changed** or **Full re-download**. A fetch uses HubSpot's overwrite behavior — the Manager blocks the fetch when that portal's local scope has uncommitted work, protecting local changes from being overwritten.
6. When work is ready, choose **Deploy**. Select the intended files and review the target portal. The Manager runs offline HubL and JSON lint and compares selected files with HubSpot. On conflict, inspect the diff and choose **Skip**, **Overwrite**, or **Cancel**. Review or edit the suggested commit message. After successful uploads, the Manager writes a scoped local Git commit.

## Daily development workflow

1. **Start clean and current.** Confirm the selected portal and resolve any outstanding local changes before editing CMS files.
2. **Choose Fetch, Watch, or Deploy.**
   - Fetch checks each saved path for remote changes; offers **Only what changed** or **Full re-download**.
   - Watch checks HubSpot before starting, offers targeted sync / full re-download / continue without syncing, then uploads saved local changes continuously.
   - Deploy checks selected files against the last synced remote state and can show the exact diff when HubSpot changed. Per conflict: **Skip**, **Overwrite**, or **Cancel**. Successful uploads are recorded in a scoped Git commit.
3. **Review the work.** Confirm only the intended portal and files changed before closing the task.

## Tips and best practices

- Before starting a new feature, run the Manager's Fetch/check flow to see if the local copy is behind HubSpot. Resolve remote changes before editing.
- Keep all work tracked in Git so rollback stays straightforward. When a task or feature is done, use Deploy to upload the selected files and create the corresponding local commit.
- This workspace is not synchronized with a remote Git repository by default — Git is used locally to keep the working tree organized, reviewable, and recoverable.
- Use Watch carefully: saved changes upload directly to the selected HubSpot portal. At the end of every Watch session, run Deploy so the latest changes are captured in a local commit.
- Run Lint before uploading. The Manager checks HubL delimiter/block balance and JSON syntax offline, and the same checks run automatically as a gate before Watch or Deploy uploads.

## Links and references

- [hs-dev-manager on npm](https://www.npmjs.com/package/hs-dev-manager)
- [HubSpot CLI installation](https://developers.hubspot.com/docs/developer-tooling/local-development/hubspot-cli/install-the-cli)
- [Node.js downloads](https://nodejs.org/en/download)
- [Git downloads](https://git-scm.com/downloads/)
