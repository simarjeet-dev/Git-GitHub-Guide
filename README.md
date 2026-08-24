# Shopify Development — Git & GitHub Guide

Internal reference for Git and GitHub workflows used by the Macnaught Group Shopify development team (LockNLube, STM, and other brand storefronts). This repo consolidates cheat sheets, SSH setup, repository workflows, and the team's branching SOP into one place so nobody has to dig through scattered PDFs to remember a command.

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Git Basics — Command Cheat Sheet](#2-git-basics--command-cheat-sheet)
  - [2.1 Setup](#21-setup)
  - [2.2 Setup & Init](#22-setup--init)
  - [2.3 Stage & Snapshot](#23-stage--snapshot)
  - [2.4 Branch & Merge](#24-branch--merge)
  - [2.5 Share & Update](#25-share--update)
  - [2.6 Tracking Path Changes](#26-tracking-path-changes)
  - [2.7 Temporary Commits (Stash)](#27-temporary-commits-stash)
  - [2.8 Rewrite History](#28-rewrite-history)
  - [2.9 Inspect & Compare](#29-inspect--compare)
  - [2.10 Ignoring Files](#210-ignoring-files)
- [3. SSH Setup for Multiple GitHub Accounts (Windows)](#3-ssh-setup-for-multiple-github-accounts-windows)
- [4. Cloning a Repository into an Existing Folder (LNL-US Example)](#4-cloning-a-repository-into-an-existing-folder-lnl-us-example)
- [5. Reconnecting a Repository After an Organization Transfer](#5-reconnecting-a-repository-after-an-organization-transfer)
- [6. Shopify Theme Git Workflow (Daily Operations)](#6-shopify-theme-git-workflow-daily-operations)
- [7. Macnaught Group Git Branching & Environment Sync SOP](#7-macnaught-group-git-branching--environment-sync-sop)
- [8. Troubleshooting Quick-Index](#8-troubleshooting-quick-index)

---

## 1. Introduction

This guide combines six previously separate references into one:

- Git Cheat Sheet (GitHub Education) — core Git command reference
- GitHub Multiple Accounts SSH Setup Guide — Windows SSH config for work + personal GitHub accounts
- LNL-US Git Clone and Switch to simarjeet-dev Guide — cloning a theme repo into an existing folder
- Shopify Theme Git Workflow Guide — daily workflow and Shopify-specific merge conflict scenarios
- Github Connect Repo After Transfer — fixing "repository not found" after an org transfer
- Macnaught Git Branching SOP — the team's four-branch (dev → staging → main) process

> Command syntax below uses Windows PowerShell conventions where relevant (as used on team laptops). Adjust path syntax if working on macOS/Linux.

## 2. Git Basics — Command Cheat Sheet

### 2.1 Setup

```bash
git config --global user.name "[firstname lastname]"   # name attached to your commits
git config --global user.email "[valid-email]"          # email attached to your commits
git config --global color.ui auto                       # automatic colour output in terminal
```

### 2.2 Setup & Init

```bash
git init          # initialize an existing folder as a Git repository
git clone [url]   # retrieve an entire repository from a hosted location
```

### 2.3 Stage & Snapshot

```bash
git status              # show modified / staged files
git add [file]          # stage a file for the next commit
git reset [file]        # unstage a file, keep changes in working directory
git diff                # show changes not yet staged
git diff --staged       # show changes staged but not yet committed
git commit -m "[message]"   # commit staged content as a new snapshot
```

### 2.4 Branch & Merge

```bash
git branch                  # list branches (* marks active one)
git branch [branch-name]    # create a new branch at the current commit
git checkout [branch]       # switch to another branch
git merge [branch]          # merge specified branch into current branch
git log                     # show commit history for current branch
git branch -d [branch]      # delete a branch (safe — won't delete if not fully merged)
git branch -D [branch]      # force delete a branch
```

### 2.5 Share & Update

```bash
git remote add [alias] [url]   # add a Git URL as a named remote
git remote -v                  # show configured remotes with their URLs
git fetch [alias]              # download all branches from a remote, no merge
git fetch --all --prune        # fetch updates from all remotes, remove deleted remote branches
git merge [alias]/[branch]     # merge a fetched remote branch into current branch
git push [alias] [branch]      # push local commits to remote branch
git pull                       # fetch and merge from the tracked remote branch
git branch -a                  # list all branches including remote branches
```

### 2.6 Tracking Path Changes

```bash
git rm [file]                  # delete a file, stage the removal
git mv [old-path] [new-path]   # move/rename a file, stage the change
git log --stat -M              # show commit logs including moved/renamed paths
```

### 2.7 Temporary Commits (Stash)

```bash
git stash          # temporarily save modified and staged changes
git stash list     # list stashed change sets in stack order
git stash pop      # reapply most recent stash and remove it from the stack
git stash drop     # discard most recent stash without applying it
```

### 2.8 Rewrite History

```bash
git rebase [branch]         # reapply current-branch commits on top of another branch
git reset --hard [commit]   # clear staging area, reset working tree to a commit
```

> ⚠️ `git reset --hard` and `rebase` rewrite history — avoid on shared branches (`staging`, `main`) unless the whole team is aligned.

### 2.9 Inspect & Compare

```bash
git log branchB..branchA     # commits on branchA not on branchB
git log --follow [file]      # commits that changed a file, even across renames
git diff branchB...branchA   # what's in branchA that isn't in branchB
git show [SHA]                # show a specific commit/object
```

### 2.10 Ignoring Files

```bash
git config --global core.excludesfile [file]   # system-wide ignore pattern file
```

Project-level ignores go in `.gitignore`:

```
logs/
*.notes
pattern*/
```

## 3. SSH Setup for Multiple GitHub Accounts (Windows)

Use this when one Windows machine needs to push to both a personal GitHub account and the Macnaught work GitHub organization without credentials colliding.

**1. Generate SSH keys**

```bash
ssh-keygen -t ed25519 -C "work-email@example.com" -f "$HOME\.ssh\id_ed25519_work"
ssh-keygen -t ed25519 -C "personal-email@example.com" -f "$HOME\.ssh\id_ed25519_personal"
```

**2. Verify keys**

```bash
ls ~/.ssh
```

**3. Create `C:\Users\<you>\.ssh\config`** (no file extension)

```
Host github-work
 HostName github.com
 User git
 IdentityFile ~/.ssh/id_ed25519_work
 IdentitiesOnly yes

Host github-personal
 HostName github.com
 User git
 IdentityFile ~/.ssh/id_ed25519_personal
 IdentitiesOnly yes
```

**4. Add public keys to GitHub** (Settings → SSH and GPG keys)

```bash
Get-Content $HOME\.ssh\id_ed25519_work.pub
Get-Content $HOME\.ssh\id_ed25519_personal.pub
```

**5. Test the connections**

```bash
ssh -T git@github-work
ssh -T git@github-personal
```

**6. Update the remote on an existing repository**

```bash
# Personal
git remote set-url origin git@github-personal:USERNAME/REPO.git
# Work
git remote set-url origin git@github-work:ORG/REPO.git
```

**7. Configure commit identity per repository**

```bash
# Personal
git config user.name "Simarjeet Singh"
git config user.email "simar8211@gmail.com"
# Work
git config user.name "Simarjeet Singh"
git config user.email "simarjeet.singh@macnaught.com"
```

**8. Verify everything**

```bash
git remote -v
git config user.name
git config user.email
git log --format=fuller -1
```

**9. Pushing a brand-new project for the first time** (no existing remote/origin)

```bash
# Personal account
git init
git add .
git commit -m "Initial commit: Shopify Theme Manager with dashboard and CLI tools"
git branch -M main
git remote add origin git@github-personal:simarjeet-dev/Shopify-Themes-Manager.git
git config user.name "Simarjeet Singh"
git config user.email "simar8211@gmail.com"
git push -u origin main
```

```bash
# Work account (swap remote + identity only)
git remote add origin git@github-work:ORG/REPO.git
git config user.name "Simarjeet Singh"
git config user.email "simarjeet.singh@macnaught.com"
git push -u origin main
```

**Notes**

- SSH keys and the SSH config are configured once per computer, not once per repo.
- Remote URL and local git identity are configured once per local repository.
- Existing repository → use `set-url` (step 6). Brand-new project → use `init` + `remote add` (step 9).

## 4. Cloning a Repository into an Existing Folder (LNL-US Example)

Worked example: cloning `LockNLube-New-Zealand` directly into an already-created, blank local folder and checking out `simarjeet-dev`. Assumes the `github-work` SSH alias from Section 3 is already configured.

```bash
cd "C:\Users\simarjeet.singh\OneDrive - Macnaught Pty Ltd\Simarjeet Singh\LockNLube\LNL-US"
git clone --branch simarjeet-dev git@github-work:Macnaught/LockNLube-New-Zealand.git .
git branch
git branch -a
git remote -v
git config user.name "Simarjeet Singh"
git config user.email "simarjeet.singh@macnaught.com"
git config user.name
git config user.email
```

**Notes**

- The trailing `.` on the clone command is required to clone directly into the existing blank folder, instead of creating a nested subfolder.
- `--branch simarjeet-dev` clones and checks the branch out in one step — no separate checkout is needed.
- If a store uses a different repository, swap the URL for the correct one.

## 5. Reconnecting a Repository After an Organization Transfer

Fixes the `Repository not found` error after a repo has been transferred from a personal account into the Macnaught GitHub organization, while the local clone still points at the old owner.

**The error**

```
remote: Repository not found.
fatal: repository 'https://github.com/simarjeet-dev/LockNLube-New-Zealand.git/' not found
```

**Fix**

```bash
git remote -v
# origin  https://github.com/simarjeet-dev/LockNLube-New-Zealand.git (fetch/push)

# get the new URL from the org repo's "Code" button, then:
git remote set-url origin https://github.com/ORG_NAME/LockNLube-New-Zealand.git

git remote -v
git fetch origin
```

> If the repo was cloned over SSH rather than HTTPS, use the org's SSH URL instead (e.g. `git@github-work:ORG/REPO.git`) — see Section 3 for SSH alias setup.

## 6. Shopify Theme Git Workflow (Daily Operations)

### Standard daily workflow

```bash
git fetch origin   # check for remote updates without touching local files
git status
```

| Status shown | Meaning | Action |
|---|---|---|
| Your branch is up to date | Local and remote are in sync | No action required |
| Your branch is behind origin/main | Remote has newer commits | `git pull origin main` |
| Your branch is ahead of origin/main | Local has unpushed commits | `git push origin main` |

### Common Shopify theme scenarios

**Scenario 1 — Shopify Admin changes auto-committed to GitHub**

Happens when Theme Customizer edits get auto-committed by Shopify's GitHub integration, making local code outdated.

```
Error: Your local changes would be overwritten by merge
```

```bash
git restore .
git pull origin main
```

**Scenario 2 — Keep local changes while pulling latest code**

```bash
git stash
git pull origin main
git stash pop
```

> If conflicts appear after `git stash pop`, resolve them manually and commit.

**Scenario 3 — Commit local changes before pulling**

```bash
git add .
git commit -m "Describe your changes"
git pull origin main
git push origin main
```

### Key commands

```bash
git add layout/theme.liquid       # stage a specific file
git restore layout/theme.liquid   # discard local uncommitted changes to a file
git restore .                     # discard ALL local uncommitted changes
git stash                         # temporarily store local changes without committing
```

### Recommended workflow

```bash
# Before starting work
git fetch origin
git status
git pull origin main

# After completing work
git add .
git commit -m "Describe changes"
git push origin main
```

### Files that commonly cause Shopify conflicts

- `templates/*.json`
- `config/settings_data.json`
- `layout/theme.liquid`

These are rewritten automatically by Shopify's Theme Customizer, so they conflict most often when the same theme is also being edited locally via the Shopify CLI.

### Best practices

- Avoid editing the same theme locally and in Shopify Admin at the same time
- Always pull the latest GitHub changes before starting development
- Commit changes regularly to reduce merge conflicts
- Use `git stash` before pulling if local changes are incomplete

## 7. Macnaught Group Git Branching & Environment Sync SOP

**Purpose:** keep `anshika-dev`, `simarjeet-dev`, `staging`, and `main` in sync, with a repeatable process for pulling production changes down into dev branches and pushing dev work up to production.

### 7.1 Environment / branch map

| Branch | Role |
|---|---|
| `anshika-dev` | Anshika's individual development branch |
| `simarjeet-dev` | Simarjeet's individual development branch |
| `staging` | Shared QA / UAT environment, tested before going live |
| `main` | Live production branch (the actual Shopify storefront) |

Anshika (Shopify Developer) works on `anshika-dev`. Simarjeet (Senior Shopify Developer) works on `simarjeet-dev` and reviews/approves all of Anshika's PRs before they merge into `staging`.

- **Forward flow (new work):** dev branch → PR → staging → QA sign-off → PR → main
- **Backward flow (sync down):** main → staging → dev branch, whenever main gets ahead (client hotfix, direct admin edit)

### 7.2 Backward flow — pulling production changes down

Always sync top-down: `main → staging → dev branch`. Never let a dev branch pull straight from `main`, skipping `staging`.

```bash
# Step 1 — update local main
git checkout main
git pull origin main

# Step 2 — update staging with latest main
git checkout staging
git pull origin staging
git merge main
git push origin staging

# Step 3 — update your dev branch with latest staging
git checkout simarjeet-dev
git pull origin simarjeet-dev
git merge staging
git push origin simarjeet-dev
```

Anshika runs the same Step 3 on `anshika-dev` after Step 2 is pushed.

> If conflicts occur during any merge: resolve conflicted files manually, then `git add .` followed by `git commit`. Do not force-push over teammates' work. If unsure, stop and confirm with the team before pushing.

### 7.3 Forward flow — shipping new development

```bash
# Step 1 — develop on your dev branch (sync with staging first)
git checkout simarjeet-dev
# make code changes
git add .
git commit -m "Describe the change"
git push origin simarjeet-dev
```

- **Step 2:** Open a PR on GitHub from the dev branch into `staging`. Add a clear description and link the ClickUp task. Simarjeet reviews and approves PRs raised by Anshika. Once approved, merge into `staging`.
- **Step 3:** QA testing happens on `staging` before anything moves to production. Bugs found are fixed on the originating dev branch, re-pushed, and re-PR'd into `staging`.
- **Step 4:** Only after QA sign-off, open a PR from `staging` into `main`. Final approval from the Senior Shopify Developer. Merging into `main` deploys to production.
- **Step 5:** Immediately after merging into `main`, run the full backward flow (7.2) so `staging`, `simarjeet-dev`, and `anshika-dev` all reflect the new production state.

### 7.4 Handling direct client edits to production

The most common cause of branches falling out of sync:

- Treat any direct client/admin edit to `main` as a hotfix — run the full backward sync (7.2) the same day it's noticed
- For urgent code hotfixes: branch `hotfix/*` off `main`, PR into `main`, then immediately backport into `staging` and both dev branches
- Where possible, restrict the client's live theme code-editing access, and route content-only changes through Online Store 2.0 sections/blocks so code and content stay separated
- Flag recurring direct-edit incidents so the access/process gap can be addressed with the client

### 7.5 Quick reference

- Sync down (main is ahead): `main → staging → dev branch`. Always top-down.
- Ship up (new work): `dev branch → PR → staging → QA → PR → main`.
- After every production release: re-sync staging and both dev branches immediately.
- Never skip staging in either direction.
- Never force-push shared branches (`staging`, `main`, or the other dev branch).

## 8. Troubleshooting Quick-Index

| Symptom / situation | Go to |
|---|---|
| "Repository not found" after a repo was transferred to the org | [Section 5](#5-reconnecting-a-repository-after-an-organization-transfer) |
| Need to push to both a personal and a work GitHub account | [Section 3](#3-ssh-setup-for-multiple-github-accounts-windows) |
| Setting up a brand-new blank folder as a fresh clone | [Section 4](#4-cloning-a-repository-into-an-existing-folder-lnl-us-example) |
| "Your local changes would be overwritten by merge" | [Section 6, Scenario 1](#6-shopify-theme-git-workflow-daily-operations) |
| Need to keep uncommitted local edits before pulling | [Section 6, Scenario 2](#6-shopify-theme-git-workflow-daily-operations) / [Section 2.7](#27-temporary-commits-stash) |
| `main` is ahead of staging / dev branches | [Section 7.2](#7-macnaught-group-git-branching--environment-sync-sop) (Backward Flow) |
| Ready to ship a feature from a dev branch to production | [Section 7.3](#7-macnaught-group-git-branching--environment-sync-sop) (Forward Flow) |
| Client edited the live theme directly in Shopify Admin | [Section 7.4](#7-macnaught-group-git-branching--environment-sync-sop) |
| Forgot exact syntax for a core Git command | [Section 2](#2-git-basics--command-cheat-sheet) |

---

*Maintained by the Macnaught Group Shopify Development team. Last updated: August 2026.*
