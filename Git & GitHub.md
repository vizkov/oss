# What Is Git and Why Does It Exist?

## The Problem Git Solves

Imagine you're writing code. You make a change, it breaks everything. You want to go back — but you already saved. Without git, you're stuck.

Now imagine 10 people editing the same codebase. Who changed what? When? Why? How do you merge everyone's work without destroying each other's code?

Git solves both problems. It's a **version control system** — it tracks every change ever made to your code, who made it, and when.

### Git vs GitHub — They Are NOT the Same Thing

This trips up everyone at the start.

- **Git** — a tool that runs on your computer. Tracks changes locally.
- **GitHub** — a website that hosts git repositories online. Lets teams collaborate.

You can use Git without GitHub. GitHub is just a remote place to store and share your Git history.

### The Core Mental Model

Think of Git like a **save system in a video game**, but for code:

- **Repository (repo)** — your project folder, tracked by Git
- **Commit** — a save point. A snapshot of your code at a moment in time
- **Branch** — a parallel timeline. You can work on something without affecting the main code
- **Merge/Rebase** — combining two timelines back together
- **Remote** — a copy of your repo hosted online (e.g. on GitHub)
- **Push** — sending your local commits to the remote
- **Pull** — getting the remote's commits onto your local machine

---

# Setup

## Install Git

Download from https://git-scm.com — comes pre-installed on Mac/Linux.

Verify:

```bash
git --version
```

## Configure Your Identity

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

Git stamps every commit with this info. Do this once.

## Set Up GitHub

1. Create account at github.com
2. Set up SSH key (so you don't type password every time):

```bash
ssh-keygen -t ed25519 -C "your@email.com"
# Press enter through all prompts
cat ~/.ssh/id_ed25519.pub
# Copy the output and paste it into GitHub → Settings → SSH Keys
```

## Useful Git Config

```bash
# Better diff output
git config --global diff.algorithm histogram

# Auto-setup remote tracking on push
git config --global push.autoSetupRemote true

# Rebase by default on pull
git config --global pull.rebase true

# Useful aliases
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.st "status -s"
git config --global alias.undo "reset HEAD~1"
```

---

# Core Concepts in Depth

## The Three Areas of Git

This is the most important thing to understand about Git's internals:

```
Working Directory  →  Staging Area  →  Repository (History)
  (your files)         (git add)        (git commit)
```

1. **Working Directory** — files as they exist on your disk right now
2. **Staging Area** — files you've marked as "ready to commit" (via `git add`)
3. **Repository** — the permanent history of all commits

A file change doesn't become a commit automatically. You explicitly choose what goes into each commit. This is intentional — it gives you control.

## What a Commit Actually Is

A commit is a **snapshot**, not a diff. Git stores the full state of every tracked file at that moment. It also stores:

- A unique ID (hash) like `a3f8c21`
- Author name and email
- Timestamp
- Commit message
- Pointer to the previous commit (parent)

This chain of commits pointing to each other is your project history.

## What a Branch Actually Is

A branch is just a **pointer to a commit**. That's it. When you create a branch, Git creates a new pointer. When you commit on that branch, the pointer moves forward.

```
main:    A → B → C
                 ↑
              (main points here)

feature: A → B → C → D → E
                         ↑
                       (feature points here)
```

Branches are cheap and fast in Git. Use them for everything.

---

# Daily Workflow

## Starting a New Project

```bash
mkdir my-project
cd my-project
git init                    # creates a .git folder — this IS the repository
```

## Cloning an Existing Project

```bash
git clone https://github.com/username/repo-name.git
cd repo-name
```

## The Everyday Loop

```bash
# 1. Check what's changed
git status

# 2. See the actual changes
git diff

# 3. Stage changes
git add filename.ts          # stage a specific file
git add .                    # stage everything

# 4. Review what's staged
git diff --staged

# 5. Commit
git commit -m "fix: brief description of what and why"

# 6. Push to remote
git push origin branch-name
```

## Checking History

```bash
git log                      # full history
git log --oneline            # compact — one line per commit
git log --oneline --graph    # visualize branches
```

---

# Branching

## Why Branch?

`main` is sacred — it should always be working, stable code. You never work directly on it. Instead, create a branch for every change, no matter how small.

## Creating and Switching Branches

```bash
# Create and switch in one command
git checkout -b fix/my-bug

# Modern syntax (Git 2.23+)
git switch -c fix/my-bug

# Switch to an existing branch
git checkout main
git switch main

# See all branches
git branch
```

## Branch Naming Conventions

- `fix/description` — bug fixes
- `feat/description` — new features
- `docs/description` — documentation only
- `refactor/description` — code cleanup
- `test/description` — test additions

Keep branches **focused** — one branch per logical change. Don't mix bug fixes with refactors in the same PR.

## Branch Off the Right Place

```bash
# Always start from the latest upstream main
git fetch upstream
git checkout upstream/main -b feat/my-feature
```

If you branch off stale code, you'll have more conflicts to deal with later.

---

# Working With Remotes

## Remote Vocabulary

- **origin** — your fork on GitHub (your copy)
- **upstream** — the original repo you forked from

```bash
# See your remotes
git remote -v

# Add upstream (do this once after cloning your fork)
git remote add upstream https://github.com/ORIGINAL/REPO.git
```

## Fork and Clone

```bash
# Fork on GitHub UI first, then:
git clone https://github.com/YOUR_USERNAME/REPO_NAME.git
cd REPO_NAME

# Add upstream remote
git remote add upstream https://github.com/ORIGINAL_OWNER/REPO_NAME.git

# Verify
git remote -v
```

## Syncing With Upstream

```bash
# Get latest changes from original repo
git fetch upstream

# Apply them to your branch (rebase preferred over merge)
git rebase upstream/main
```

Do this regularly — don't let your branch drift too far behind.

**Rebase vs Merge:** Always rebase for OSS contributions. It produces a linear history and avoids merge commits that pollute the log.

If you get conflicts during rebase:

```bash
# Fix conflicts in files, then:
git add .
git rebase --continue

# If things go wrong:
git rebase --abort
```

## Push and Pull

```bash
git push origin branch-name      # send your commits to GitHub
git pull origin main             # get latest + merge into current branch
git fetch origin                 # get latest WITHOUT merging
```

**Fetch vs Pull:** `fetch` is safe — it just downloads, doesn't change your files. `pull` = fetch + merge. When in doubt, fetch first and look before merging.

---

# Committing

## Atomic Commits

Each commit should do **one thing**. If you can't describe a commit in one sentence without "and", split it.

```bash
# Bad
git commit -m "fix uri bug and update tests and clean up comments"

# Good
git commit -m "fix: normalize URI drive letter casing in map lookup"
```

## Commit Message Format (Conventional Commits)

Most OSS projects follow this standard:

```
<type>(<scope>): <short description>

<optional body — explain WHY, not WHAT>

<optional footer — issue references>
```

Types:

- `fix` — bug fix
- `feat` — new feature
- `refactor` — no behavior change
- `docs` — documentation
- `test` — tests only
- `chore` — build/tooling

Example:

```
fix(uri): normalize drive letter casing for Windows paths

workspace.findFiles() returns URIs with lowercase drive letters
while onDidOpenTextDocument returns uppercase. This caused map
lookups to fail since Uri objects use reference equality.

Fixed by comparing via .path with drive letter lowercased.

Closes #1234
```

## Check Before Committing

```bash
git diff                    # see unstaged changes
git diff --staged           # see staged changes
git status                  # overall state
```

---

# Cleaning Up Commits Before a PR

This is where beginners get confused. You've been working, making lots of small commits including debug logs, typo fixes, "WIP" commits. Before opening a PR, you want to present clean, professional history.

## Interactive Rebase — Your Best Friend

```bash
git rebase -i upstream/main
```

This opens an editor showing all your commits since branching from main:

```
pick a1b2c3 fix: normalize URI comparison
pick d4e5f6 add debug console.log
pick g7h8i9 remove debug log
pick j0k1l2 fix typo in comment
```

Change `pick` to:

- `reword` — keep commit, edit message
- `squash` / `s` — merge into previous commit, combine messages
- `fixup` / `f` — merge into previous commit, discard message
- `drop` / `d` — delete commit entirely
- `edit` / `e` — pause to amend the commit

Example — squashing debug commits:

```
pick a1b2c3 fix: normalize URI drive letter casing
fixup d4e5f6 add console.log for debugging       # squashed
fixup g7h8i9 remove debug logs                   # squashed
reword j0k1l2 add unit test for URI normalization # keep but rename
```

Result: one clean commit instead of four messy ones.

## Amending the Last Commit

```bash
# Fix last commit message
git commit --amend -m "fix: better message"

# Add forgotten file to last commit
git add forgotten-file.ts
git commit --amend --no-edit
```

## Force Push (after rebase/amend)

```bash
# Required after rewriting history
# --force-with-lease is safer than --force: fails if someone else pushed to your branch
git push origin fix/uri-normalization --force-with-lease
```

---

# Pull Requests

## Mindset

OSS maintainers are volunteers. Your job as a contributor is to make their life easier: clean commits, clear descriptions, minimal noise. A PR that's hard to review is a PR that gets ignored.

## The PR Flow

```
Your Fork (origin)                Original Repo (upstream)
──────────────────                ───────────────────────
main ←──────────────── synced ────── main
  │
  └── fix/my-bug branch
        │
        └── push → open PR → review → merge
```

## Before Opening a PR

- [ ] Read the project's `CONTRIBUTING.md`
- [ ] Run the full test suite locally
- [ ] Check code style matches the project (linting, formatting)
- [ ] Rebase on latest upstream main
- [ ] Clean up commits (no debug logs, no WIP commits)
- [ ] Self-review your own diff on GitHub
- [ ] Open an issue first for large changes — check if maintainer wants it

## Opening a PR

1. Push your branch to origin: `git push origin fix/my-bug`
2. Go to GitHub — you'll see a banner "Compare & pull request"
3. Fill in title and description
4. Submit

## PR Title and Description

Title: same format as a commit message — `fix(uri): normalize drive letter casing`

Description template:

```markdown
## What
Brief description of the change.

## Why
What problem does this solve? Link to issue if applicable.

## How
Key implementation decisions. Anything non-obvious the reviewer should know.

## Testing
How you verified this works. Screenshots if UI change.

Fixes #1234
```

## PR Size

Keep PRs small and focused. A PR that touches 20 files is hard to review and slow to merge. If your change is large, discuss with maintainers first or split into sequential PRs.

## Responding to Review

- Address every comment — either make the change or explain why not
- Don't resolve reviewer comments yourself — let the reviewer do it
- After making changes, re-request review explicitly
- Reply to every comment, even if just "done" or "good point, fixed"

## Keeping Your PR Up to Date

While your PR is open, upstream main might get new commits. Keep yours current:

```bash
git fetch upstream
git rebase upstream/main
git push origin fix/my-bug --force-with-lease
```

---

# Common Scenarios

## Undo Local Changes

```bash
git restore filename.ts          # discard unstaged changes (modern)
git checkout -- filename.ts      # same (old syntax)
git restore --staged filename.ts # unstage a file
git reset --hard upstream/main   # nuclear — discard everything back to upstream
```

## Undo Commits (Before Push)

```bash
git reset HEAD~1                 # undo last commit, keep changes unstaged
git reset --soft HEAD~1          # undo last commit, keep changes staged
git reset --hard HEAD~1          # undo last commit, discard changes (dangerous)
```

## Revert a Pushed Commit

```bash
# Creates a new "undo" commit — safe after pushing
git revert <commit-hash>
```

**Rule of thumb:** `reset` rewrites history (use before pushing), `revert` adds an undo commit (safe after pushing).

## Cherry-Pick a Commit

```bash
# Apply a specific commit from another branch
git cherry-pick <commit-hash>
```

## Stash Work in Progress

```bash
git stash                        # save current changes
git stash pop                    # restore them
git stash list                   # see all stashes
git stash drop                   # discard top stash
```

## Find What Broke Something

```bash
git bisect start
git bisect bad                   # current commit is broken
git bisect good v1.0.0           # this version was fine
# Git checks out midpoint — test it, then:
git bisect good                  # or: git bisect bad
# Repeat until Git identifies the culprit commit
git bisect reset                 # exit bisect mode
```

---

# Undoing Things

```bash
# Discard changes to a file (not yet staged)
git restore filename.ts

# Unstage a file (staged but not committed)
git restore --staged filename.ts

# Undo last commit, keep changes (staged)
git reset --soft HEAD~1

# Undo last commit, keep changes (unstaged)
git reset HEAD~1

# Undo last commit, discard changes (dangerous — gone forever)
git reset --hard HEAD~1

# Discard ALL local changes and go back to last commit
git reset --hard HEAD

# Revert a commit that's already been pushed (creates a new "undo" commit)
git revert <commit-hash>
```

---

# Etiquette and Unwritten Rules

- **Do** open an issue before a large PR. Maintainers may already be working on it, or may not want the change.
- **Do** check if there's an existing PR for your fix before opening one.
- **Do** be patient. Maintainers are volunteers. Follow up politely after 1-2 weeks if no response.
- **Don't** take review feedback personally. It's about the code.
- **Don't** argue with maintainers about style preferences in their repo.
- **Don't** push directly to main even if you accidentally have write access.
- **Don't** open draft PRs unless you want early feedback — use them intentionally.

---

# The OSS Contribution Checklist

Before opening a PR on any open source project:

- [ ] Read `CONTRIBUTING.md` in the repo (if it exists)
- [ ] Open an issue first for large changes — check if maintainer wants it
- [ ] Branch off latest upstream main
- [ ] Write clean, focused commits (no debug logs, no WIP)
- [ ] Run tests locally — don't break the build
- [ ] Match the project's code style
- [ ] Self-review your diff on GitHub before submitting
- [ ] Write a clear PR description
- [ ] Be patient — maintainers are volunteers

---

# Quick Reference Card

|What you want to do|Command|
|---|---|
|Initialize a repo|`git init`|
|Clone a repo|`git clone <url>`|
|See what changed|`git status`|
|See the diff|`git diff`|
|Stage a file|`git add filename`|
|Stage everything|`git add .`|
|Commit|`git commit -m "message"`|
|Push|`git push origin branch`|
|Create branch|`git checkout -b branch-name`|
|Switch branch|`git checkout branch-name`|
|See all branches|`git branch`|
|See history|`git log --oneline`|
|Sync from upstream|`git fetch upstream && git rebase upstream/main`|
|Clean up commits|`git rebase -i upstream/main`|
|Undo last commit|`git reset HEAD~1`|
|Discard file changes|`git restore filename`|
|Force push safely|`git push --force-with-lease`|
|Stash changes|`git stash`|
|Apply stash|`git stash pop`|
|Find breaking commit|`git bisect start`|