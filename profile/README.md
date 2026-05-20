# Git dev hygiene

A practical guide for **getting set up and saving work safely** with Git and GitHub. You do not need to be a developer — follow the sections **in order** the first time.

| Step | Section | What you will do |
|------|---------|------------------|
| 1 | [Terminal setup](#1-terminal-setup) | Install Git, choose macOS / PowerShell / WSL, connect to GitHub |
| 2 | [Using an IDE](#2-using-an-ide) | Edit files in VS Code, PyCharm, Cursor, etc. |
| 3 | [Typical Git workflow](#3-typical-git-workflow) | Day-to-day: branch → edit → commit → push → open a PR |
| 4 | [More detail](#4-more-detail) | Deeper rules, edge cases, and troubleshooting |

**Git in one sentence:** your project folder plus a **history of saved snapshots** (commits) you share via GitHub.

---

## 1. Terminal setup

Git commands are the same everywhere; what changes is **which terminal app you use** and **where folders live on disk**. Pick one setup per project and stick to it — mixing Windows Git and WSL Git on the same folder causes confusing errors and line-ending issues.

### Install Git and set your identity

Do this once after installing Git:

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Use a work email if your org requires it. Check with `git config --global --list`.

### macOS (Terminal / iTerm)

Git often ships with the Xcode Command Line Tools. Install or update:

```bash
# Option A — Apple CLI tools (prompts a GUI installer if missing)
xcode-select --install

# Option B — Homebrew (newer Git, easier to upgrade)
brew install git
```

Open **Terminal** (or iTerm2). Keep project folders under your home directory, e.g. `~/projects/`:

```bash
mkdir -p ~/projects
cd ~/projects
```

**SSH key for GitHub (recommended):**

```bash
ssh-keygen -t ed25519 -C "you@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub   # add this key in GitHub → Settings → SSH keys
ssh -T git@github.com
```

### Windows (PowerShell)

Install **Git for Windows** (includes `git`, Git Bash, and credential helpers):

```powershell
# Option A — winget
winget install --id Git.Git -e --source winget

# Option B — installer: https://git-scm.com/download/win
```

Open **PowerShell** or **Windows Terminal** → PowerShell. Keep project folders under e.g. `C:\Users\You\projects\`:

```powershell
New-Item -ItemType Directory -Force -Path "$HOME\projects"
cd $HOME\projects
```

**Line endings on Windows** — avoids odd line-break changes in shared repos:

```powershell
git config --global core.autocrlf input
```

**SSH key in PowerShell:**

```powershell
ssh-keygen -t ed25519 -C "you@example.com"
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
# add to GitHub → Settings → SSH keys
ssh -T git@github.com
```

Use HTTPS remotes if SSH is blocked on your network; SSH is preferred when it works.

### WSL (Windows Subsystem for Linux)

WSL is a Linux environment inside Windows. Use it when your team expects Linux-style tooling. Do Git work **inside WSL**, not on files under `C:\` opened as `/mnt/c/...`.

**Install WSL and Ubuntu** (PowerShell as Administrator):

```powershell
wsl --install
# reboot if prompted; first launch creates a Linux user/password
```

Older Windows or manual distro install:

```powershell
wsl --install -d Ubuntu
wsl --list --verbose
```

**Inside the WSL terminal** (Ubuntu):

```bash
sudo apt update
sudo apt install -y git

git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

mkdir -p ~/projects
cd ~/projects
```

**SSH key inside WSL** (separate from Windows — create keys in WSL if that is where you work daily):

```bash
ssh-keygen -t ed25519 -C "you@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
ssh -T git@github.com
```

### Which terminal should I use?

| You are on… | Open this | Put projects in… |
|-------------|-----------|------------------|
| Mac | Terminal / iTerm | `~/projects/` |
| Windows (no WSL) | PowerShell or Git Bash | `%USERPROFILE%\projects\` |
| Windows + WSL | Ubuntu in WSL | `~/projects/` inside WSL |

**Rules of thumb**

- Always run `git status`, `git commit`, and `git push` in the **same** terminal environment for that folder.
- Do not open the same repo with both Windows Git and WSL Git unless someone on your team has set that up deliberately (usually avoid it).
- If every file looks “changed” after you save, ask about line endings (`core.autocrlf`) and whether you are editing from two environments.

---

## 2. Using an IDE

Use your editor (VS Code, PyCharm, Cursor, or similar) to **open and edit files** — search, formatting, previews, and extensions. Use the **terminal** (from section 1) for Git commands in [section 3](#3-typical-git-workflow).

- **Edit in the IDE; save to Git in the terminal.** That keeps what you upload clear and matches what teammates see on GitHub.
- **Glance at changed-file highlights and the branch name** in the IDE so you know what you are working on — then run `git status` in the terminal before you commit.
- **On Windows with WSL:** in VS Code / Cursor, use **“Reopen Folder in WSL”** so the terminal path matches `~/projects`, not `C:\Users\...`.
- **Do not commit IDE-only clutter** (`.idea/`, personal `.vscode/` settings, caches) unless the repo says to share them — see [Repository hygiene](#repository-hygiene).

If you fix a merge conflict in the editor, remove all `<<<<<<<` / `=======` / `>>>>>>>` markers, save, then run `git add` and `git merge --continue` or `git rebase --continue` in the terminal. If the IDE and `git status` disagree, trust the terminal.

---

## 3. Typical Git workflow

This is the loop you will use most days. Run these commands in the terminal where you set up Git (section 1).

### First time: get a copy of the repo

```bash
cd ~/projects          # or your projects folder from section 1
git clone git@github.com:org/repo.git
cd repo
```

### Each task: branch, edit, commit, push

```bash
# Start from the main line of the project
git checkout main          # or: git switch main
git pull origin main

# Create a branch for your changes (use a short descriptive name)
git checkout -b docs/update-readme   # or: git switch -c docs/update-readme

# ... edit files in your IDE ...

# See what changed
git status
git diff                   # unstaged changes
git diff --staged          # staged changes

# Stage and save a snapshot (commit)
git add .                  # all changes — use `git add path/to/file` when you want only some files
git add path/to/file.md
git commit -m "Update readme with onboarding steps"

# Upload your branch (first time on this branch)
git push -u origin docs/update-readme
git push                   # on later commits, after upstream is set
```

### Open a pull request (PR)

On GitHub, open a **Pull request** from your branch (e.g. `docs/update-readme`) into `main`. That is how you ask the team to review and merge your work. Add a short description of **what you changed and why**.

### Quick checklist before you ask for review

- [ ] `git pull` on `main` or rebased your branch so you are not behind the team
- [ ] `git status` shows only what you mean to include — no secrets or stray files
- [ ] Commit message and PR text explain the change in plain language
- [ ] `git push` succeeded; PR is open on GitHub

---

## 4. More detail

The sections below are **reference** for doing things well and fixing mistakes. You do not need to read them all on day one.

### Commits

- **One logical change per commit.** Split unrelated edits so history is easy to follow and undo.
- **Write messages for humans.** Short title: what you did (`Update readme`, not `Updated readme`). Add a second line when *why* matters.
- **Save often on your branch.** Small commits are easier to review than one huge dump at the end.
- **Never commit secrets** (passwords, API keys, `.env` files). If one slips in, tell the team immediately.

```bash
git commit -m "Fix typo in onboarding doc"
git commit -m "Fix typo in onboarding doc" -m "Caught during dry run with new hires."
git add -p path/to/file.md    # stage only part of a file (advanced)
```

### Branches

- **Branch from `main`** unless someone tells you otherwise.
- **Use clear names:** `docs/fix-typo`, `feat/export-csv`, `fix/broken-link`.
- **Delete merged branches** when prompted — less clutter.

```bash
git checkout main
git pull origin main
git checkout -b docs/fix-typo
git branch
git checkout docs/fix-typo
git branch -d docs/fix-typo
git push origin --delete docs/fix-typo
```

### Before you push

- **Update from `main` first** so you are not surprised by conflicts on GitHub.
- **Skim your own diff** before pushing.

```bash
git fetch origin
git checkout main
git pull origin main
git checkout docs/fix-typo
git merge main              # or: git rebase main (team preference)
git log --oneline -5
git diff main...HEAD
```

### Pull requests

- **One PR, one purpose.** Split large unrelated changes.
- **Say what and why** in the PR description — and how someone can check your work.
- **Reply to review comments** and push updates with `git add`, `git commit`, `git push`.

```bash
git add .
git commit -m "Address review: clarify install steps"
git push
```

### History and rewriting

- **Do not force-push `main`** or shared branches.
- **Force-push only your own branch** when you know no one else is using it — and only after a rebase/amend.

```bash
git commit --amend -m "Updated message"    # last commit not pushed yet
git push --force-with-lease                # your feature branch only
```

### Conflicts and merges

When Git cannot combine changes automatically, it marks files for you to fix manually.

```bash
git merge main
# fix files in IDE, remove conflict markers, save
git add path/to/resolved-file.md
git merge --continue
git merge --abort    # start over if needed
```

### Repository hygiene

- **Respect `.gitignore`** — do not add build output, dependencies, or personal IDE files unless the repo expects them.
- **Do not skip git hooks** with `--no-verify` unless the team agrees you should.

```bash
git restore --staged path/to/file.md
git restore path/to/file.md
git check-ignore -v path/to/file
```

### Security and access

- **Use SSH keys or tokens** with least access; do not share passwords in the repo.
- **Report leaked credentials** — revoke, rotate, then tell the team.

```bash
git remote -v
```

### When something goes wrong

| Situation | Commands |
|-----------|----------|
| Wrong files staged | `git restore --staged <file>` |
| Unstage everything | `git restore --staged .` |
| Fix last commit (not pushed) | `git commit --amend` |
| Undo last commit (already on GitHub) | `git revert HEAD` then `git push` |
| Discard all local edits | `git restore .` |
| “Lost” work | `git reflog` then recover the listed commit |

```bash
git reflog
git checkout -b recover-work abc1234
git revert abc1234
git push
```

---

*Living guide — each repo may have extra rules; when in doubt, ask your team.*
