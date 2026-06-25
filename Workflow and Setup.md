# Environment Setup

## Machine Setup

### Docker Per Project

Docker is better than a VM for this use case — lighter, faster, and you spin up/tear down per project instantly without managing a full OS. 

```bash
# Clone the target repo
git clone https://github.com/target/project
cd project

# If they provide a docker-compose.yml — use it directly
docker-compose up

# If not — spin up a container matching their stack
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  node:20 bash       # swap for python:3.11, php:8.2, etc.

# Inside the container
npm install
npm start
```

The `-v $(pwd):/workspace` flag mounts your local repo into the container — edit files in VS Code on your host, changes reflect instantly inside the container. Runtime is isolated, editor stays native.

Install the VS Code **Dev Containers** extension to open a repo inside Docker directly from VS Code with full editor support.

**When Docker doesn't work:** VS Code extensions, desktop apps, and GUI tools can't run in Docker. For those, run directly on your machine in an isolated directory.

### Keep Research Isolated Per Target

```
~/research/
  target-project-name/
    repo/           ← cloned source
    notes.md        ← your audit notes
    findings.md     ← confirmed vulns
    poc/            ← proof of concept scripts
    correspondence/ ← email threads with maintainers
```

## Editor Setup — VS Code for Code Auditing

VS Code is the best editor for reading unfamiliar codebases. Configure it for security research specifically.

### Essential Extensions

**Navigation & Reading:**

- **GitLens** — see who wrote every line, when, and why. Blame view is invaluable for understanding intent.
- **Code Tour** — annotate codebases with notes as you audit
- **Bookmarks** — mark interesting lines to come back to
- **Todo Tree** — surfaces all TODO/FIXME/HACK comments — developers leave breadcrumbs

**Security Specific:**

- **Semgrep** (official extension) — runs static analysis inline as you read
- **DevSkim** (Microsoft) — highlights insecure code patterns inline
- **Snyk Security** — flags vulnerable dependencies and insecure code inline

**Language Support:**

- Install the relevant language extension for whatever you're auditing (Python, PHP, Go, etc.)
- **REST Client** — send HTTP requests directly from VS Code (`.http` files)

### Useful VS Code Settings for Auditing

```json
// settings.json
{
  "editor.minimap.enabled": true,          // see file structure at a glance
  "editor.renderWhitespace": "all",        // spot weird whitespace tricks
  "editor.wordWrap": "off",               // don't miss long lines
  "search.exclude": {
    "**/node_modules": false,             // include node_modules in search
    "**/vendor": false                    // include vendor in search
  },
  "search.useIgnoreFiles": false          // don't respect .gitignore in search
}
```

Disabling gitignore/node_modules exclusion in search is important — vulnerabilities often hide in dependency code.

---

# Using Git during Security Research

## Core Principle: Private by Default

Your research repo contains PoCs, vuln details, and exploitation notes. **Never put this on a public GitHub repo.** Use a local git repo or a private remote.

```bash
# Initialize a private research repo
cd ~/research
git init
# If you want a remote backup — use a private GitHub/GitLab repo only
git remote add origin git@github.com:YOU/security-research-private.git
```

## Repository Structure

One repo for all research, organized by target:

```
~/research/
  .git/
  .gitignore
  targets/
    project-name-1/
      notes.md
      findings.md
      poc/
      correspondence/
    project-name-2/
      ...
  templates/
    findings-template.md
    disclosure-email-template.md
```

## .gitignore for Research Repos

```gitignore
# Never commit these
*.env
*credentials*
*secrets*

# Keep repo clean
node_modules/
__pycache__/
*.pyc
.DS_Store

# Exclude PoC scripts until after disclosure (safety net)
targets/*/poc/
```

## Commit Strategy

Commit often, with clear context. You'll come back to notes weeks later and need to know exactly what you were thinking.

```bash
# Starting a new target
git commit -m "research(project-name): initial recon and setup"

# During audit
git commit -m "research(project-name): suspect path traversal in upload handler L142"
git commit -m "research(project-name): confirmed XSS in search param - has PoC"
git commit -m "research(project-name): ruled out SQLi in login - input is parameterized"

# After disclosure
git commit -m "research(project-name): disclosed - awaiting maintainer response"
git commit -m "research(project-name): fix merged, CVE requested"
```

Use a `research(target):` prefix — makes the log scannable when you're juggling multiple targets.

## Branching Per Target

```bash
# Start a new investigation
git checkout -b research/project-name

# Work freely — commit notes, findings, suspects
# When closed (disclosed or ruled out) — merge back
git checkout main
git merge research/project-name
git branch -d research/project-name
```

`main` becomes your clean record of completed work. Branches are active workspaces.

## Useful Log Commands

```bash
# All active research branches
git branch | grep research/

# Recent activity across all targets
git log --oneline --all --since="2 weeks ago"

# Everything on a specific target
git log --oneline -- targets/project-name/
```

## Tags — Marking Milestones in Your Research

Tags let you mark significant moments in your research timeline — disclosure sent, fix released, CVE assigned. Unlike branches, tags are permanent markers that don't move.

```bash
# Tag when you disclose
git tag disclosed/project-name-2024-01-15

# Tag when fix ships
git tag fixed/project-name v1.2.3-patched

# Tag with a message (annotated tag — preferred)
git tag -a disclosed/project-name -m "Disclosed XSS in search param. Awaiting response."
git tag -a cve/CVE-2024-12345 -m "CVE assigned. Fix shipped in v2.1.0"

# See all tags
git tag

# See tags with messages
git tag -n

# Push tags to your private remote
git push origin --tags
```

Useful for quickly jumping back to the state of your research at a specific milestone:

```bash
# See what your notes looked like at disclosure time
git show disclosed/project-name
git checkout disclosed/project-name -- targets/project-name/findings.md
```

## Semantic Versioning — Reading Target Project Versions

When auditing a target, understanding their versioning tells you which versions are affected and how serious the maintainers are about stability.

Semantic versioning format: `MAJOR.MINOR.PATCH` — e.g. `2.4.1`

|Segment|Changes when...|Security implication|
|---|---|---|
|`MAJOR`|Breaking API changes|Old integrations may not get backported fixes|
|`MINOR`|New features, backwards compatible|Check changelog for new attack surface|
|`PATCH`|Bug fixes only|Security fixes usually land here — watch these|

```bash
# Check what version you're auditing
cat package.json | grep '"version"'
git tag --sort=-version:refname | head -10   # latest releases first

# Compare two versions to see what changed between them
git diff v2.3.0 v2.4.1 -- src/auth/

# Check if a fix was backported to older versions
git log --oneline v1.x..v2.x -- src/vulnerable-file.js
```

When reporting a vuln, always specify exact affected versions and whether older major versions are also affected — maintainers need this for their advisory.


## git stash — Switching Between Targets Without Committing

When you're mid-investigation on one target and need to urgently switch to another, stash lets you shelve your uncommitted work cleanly.

```bash
# Shelve current work on project-a
git stash push -m "project-a: mid-investigation, tracing upload handler"

# Switch to another target's branch
git checkout research/project-b

# Work on project-b...

# Come back to project-a later
git checkout research/project-a
git stash pop                         # restore where you left off

# See all stashes across all targets
git stash list
# stash@{0}: On research/project-a: project-a: mid-investigation, tracing upload handler
# stash@{1}: On research/project-c: project-c: draft findings not ready to commit
```

Always name your stashes (`-m`) — unnamed stashes are impossible to identify days later.

## git worktree — Audit Multiple Targets Simultaneously

Normally you can only have one branch checked out at a time. `git worktree` lets you check out multiple branches into separate directories at the same time — no stashing, no context switching.

```bash
# Check out project-b into a separate directory while staying on project-a
git worktree add ../research-project-b research/project-b
git worktree add ../research-project-c research/project-c

# Now you have separate directories per target — open each in its own VS Code window
# ~/research/          ← project-a (main worktree)
# ~/research-project-b ← project-b
# ~/research-project-c ← project-c

# See all active worktrees
git worktree list

# Remove when done
git worktree remove ../research-project-b
```

Best used when juggling 2-3 active targets — each gets its own directory and VS Code window with no interference.

## git grep — Search Across Your Own Research Notes

Useful when you remember writing something about a pattern weeks ago but can't find which target's notes it was in.

```bash
# Search across all current notes/findings
git grep "prototype pollution" -- targets/

# Search across ALL commits — finds things you wrote and later deleted
git grep "prototype pollution" $(git rev-list --all) -- targets/

# Find every time you noted a suspect finding
git grep "SUSPECT" -- targets/*/notes.md

# Find all confirmed vulns across all targets
git grep "confirmed" -- targets/*/findings.md
```

## git reflog — Recover Accidentally Deleted Work

If you accidentally delete a branch, hard reset, or lose uncommitted work, `reflog` is your safety net. It records every action git has taken, even ones that aren't in the normal log.

```bash
# See everything git has done recently
git reflog

# Output looks like:
# a3f8c21 HEAD@{0}: checkout: moving from research/project-a to main
# d4e5f6b HEAD@{1}: commit: research(project-a): confirmed XSS - has PoC
# g7h8i9c HEAD@{2}: branch: deleted research/old-target

# Recover a deleted branch
git checkout -b research/old-target g7h8i9c

# Recover a lost commit after a bad reset
git reset --hard d4e5f6b
```

Reflog entries expire after 90 days by default — don't wait too long to recover something.

## git shortlog — Summarize Your Research Activity

Useful for reviewing what you've worked on over a period, or generating a summary of findings for your track record.

```bash
# Summary of your commits grouped by target prefix
git shortlog --all --no-merges

# Filter to a date range — e.g. last month's work
git shortlog --all --since="1 month ago" --no-merges

# Count commits per target branch
git shortlog --all --numbered --summary

# Timeline of all disclosures and closures
git log --all --oneline --grep="disclosed\|CVE\|fix merged" --format="%ad %s" --date=short
```

## Git Config for Security Research

Configure git to make your research workflow cleaner and safer.

```bash
# Better log output — see branch graph at a glance
git config --global alias.lg "log --oneline --graph --decorate --all"

# Scoped log — activity on a specific target
git config --global alias.target "log --oneline --all --"
# Usage: git target targets/project-name/

# Show tags inline in log
git config --global log.decorate true

# Safer default — never force push without thinking
git config --global alias.fpush "push --force-with-lease"

# Better diff — highlights which words changed, not just lines
git config --global diff.algorithm histogram
git config --global diff.colorMoved zebra      # highlights moved blocks differently

# Sign commits with GPG (important if your research identity matters)
git config --global commit.gpgsign true
git config --global user.signingkey YOUR_GPG_KEY_ID

# Prevent accidental pushes to public remotes
# Set your research remote as private-only
git config --global alias.safe-push "push origin"   # always explicit
```

### Useful Output Formats

```bash
# Compact log with dates — useful for research timelines
git log --format="%h %ad %s" --date=short

# Show only your research commits across all targets
git log --format="%h %s" --author="Your Name" --all

# List all tags with dates — full disclosure history
git tag -l --sort=-creatordate --format="%(creatordate:short) %(refname:short) %(contents:subject)"

# See full diff of a finding commit
git show <hash> --stat          # summary of files changed
git show <hash> -p              # full patch
```

## Read Git History

```bash
# Look for security-related commits
git log --oneline --all | grep -i "security\|vuln\|fix\|auth\|sanitize\|escape\|inject"

# See what changed in a suspicious commit
git show <commit-hash>

# See history of a specific sensitive file
git log -p -- src/auth/login.js

# Find deleted code (sometimes vulns are "fixed" by deletion, not properly)
git log --diff-filter=D --name-only
```

GitLens in VS Code makes this workflow much smoother — you can see blame inline and browse history per file without leaving the editor.

---

# Confirming and Documenting a Vulnerability

Before reporting, be thorough. A vague report wastes everyone's time.

## Confirm It's Real

- Reproduce it consistently
- Understand the full impact — what can an attacker actually do?
- Determine if it's exploitable in a realistic scenario

## Write a Clear PoC (Proof of Concept)

```markdown
## Vulnerability: Reflected XSS in search parameter

**Severity:** High
**Affected version:** 2.3.1 (latest)
**Component:** /search endpoint, `q` parameter

### Description
The `q` parameter in the search endpoint is reflected into the HTML
response without sanitization, allowing arbitrary script execution.

### Steps to Reproduce
1. Start the application locally
2. Navigate to: http://localhost:3000/search?q=<script>alert(1)</script>
3. Observe the alert dialog executing

### Impact
An attacker can execute arbitrary JavaScript in the context of a
victim's browser session by tricking them into clicking a crafted link.
This allows session hijacking, credential theft, or account takeover.

### Suggested Fix
Encode user input before reflecting it in HTML responses. Use a
library like DOMPurify (client-side) or he/entities (server-side).
```

---

# Responsible Disclosure

## What You're Doing

You are finding vulnerabilities in software you did not write, then reporting them to the people who did. Done right, this is respected and valuable. Done wrong, it's legally risky and burns bridges with maintainers.

### The Rules

- **Never test on production systems you don't own.** Always run the software locally.
- **Never exploit a vulnerability beyond what's needed to confirm it exists.**
- **Never publicly disclose before giving maintainers time to fix** — typically 90 days (Google Project Zero standard).
- Check if the project has a **bug bounty program** — HackerOne, Bugcrowd, or their own program. Follow their rules exactly.
- Check the project's **security policy** — usually `SECURITY.md` in the repo root or on their website.

## Find the Reporting Channel

**In order of preference:**

1. `SECURITY.md` in the repo — follow their process exactly
2. `security@projectname.com` or similar email
3. GitHub's private vulnerability reporting (repo → Security tab → "Report a vulnerability")
4. Their bug bounty platform (HackerOne, Bugcrowd)

**Never open a public GitHub issue for a security vulnerability.**

## Git Workflow 

```bash
# Fork and clone
git clone https://github.com/YOUR_USERNAME/target-repo
git remote add upstream https://github.com/ORIGINAL/target-repo

# Branch name — keep it vague, don't reveal the vuln publicly
git checkout -b fix/input-sanitization

# Fix, commit cleanly
git add .
git commit -m "fix: sanitize user input before HTML reflection"

# Coordinate timing with maintainer BEFORE pushing publicly
# They may want the PR open only after a patched version is released
git push origin fix/input-sanitization
```

## Send the Report

Use encrypted email if they provide a PGP key. Otherwise plain email is fine for most OSS.

**Include:**

- Clear description of the vulnerability
- Affected versions
- Step-by-step reproduction
- Your PoC
- Impact assessment
- Suggested fix if you have one
- Your disclosure timeline (typically 90 days)

## Follow Up Timeline

- No response in **7 days** → follow up once
- No response in **14 days** → escalate (try another contact, GitHub security advisory)
- No response in **90 days** → warn them, then disclose publicly

## The Fix

**Offer to:**

- Submit a PR with the fix
- Review their proposed fix before release
- Be credited in the security advisory

## CVE

After fix is released, either the maintainer requests a CVE or you can at cveform.mitre.org. CVEs are public record of your research work.

---

## Before Submitting a Fix PR Upstream

Your public PR must be completely clean — no vuln details, no PoC, no research notes. Only the fix.

```bash
# Fork and clone the TARGET repo separately — not your research repo
git clone https://github.com/YOU/target-project
git remote add upstream https://github.com/ORIGINAL/target-project

# Branch name — vague, reveals nothing about the vuln
git checkout -b fix/input-sanitization

# Commit message: clean, no internal references
git commit -m "fix: sanitize file path input before filesystem operations"

# Coordinate timing with maintainer before pushing publicly
git push origin fix/input-sanitization
```

Never reference your private research repo, PoC, or internal notes in the public PR.

---

# Building a Track Record

## Keep a Private Research Log

```
/research
  /target-project-name
    /recon-notes.md
    /findings.md
    /poc/
    /correspondence/
```

## Why It Matters

Published CVEs and GitHub Security Advisories are permanent public record. They matter for security job applications, bug bounty reputation, and credibility for future disclosures.

## The Research Loop

```
Pick target → Read code → Static analysis → Dynamic testing
      ↓
Find something → Confirm → Document PoC
      ↓
Find SECURITY.md → Report privately → Wait
      ↓
Coordinate fix → Optional: submit PR → Coordinated public disclosure
      ↓
CVE published → Write it up → Repeat
```

---
