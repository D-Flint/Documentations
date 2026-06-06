================================================================================
                              GIT DETAILED GUIDE
================================================================================

TABLE OF CONTENTS
───────────────────────────────────────────────────────────────────────────────
1. What is Git?
2. Installation
3. Configuration
4. Branching Strategies
5. Merging vs Rebasing
6. Resolving Merge Conflicts
7. Stashing
8. Tagging
9. .gitignore Patterns
10. Remote Management
11. Pull Requests Workflow
12. git bisect
13. git reflog
14. Common Troubleshooting

================================================================================
1. WHAT IS GIT?
================================================================================

Git is a distributed version control system (DVCS) created by Linus Torvalds in
2005 for the Linux kernel development. Unlike centralized systems (SVN, CVS),
every developer's working copy is a full-fledged repository with complete history.

Key concepts:

  • Repository (repo): A directory tracked by Git, containing the .git/ folder.
  • Commit: A snapshot of changes at a point in time, identified by a SHA-1 hash.
  • Branch: A movable pointer to a commit. The default branch is main (formerly master).
  • HEAD: A pointer to the current commit your working directory is based on.
  • Working Tree: The actual files on disk that you can edit.

================================================================================
2. INSTALLATION
================================================================================

2.1 Windows
───────────────────────────────────────────────────────────────────────────────
Option A — Official installer:
  1. Download from https://git-scm.com/download/win
  2. Run the installer. Key settings:
     - Use "Git from the command line and also from 3rd-party software"
     - Use bundled OpenSSH
     - Checkout as-is, commit Unix-style line endings
     - Use MinTTY (default terminal)
     - Enable file system caching
  3. Verify:  git --version

Option B — Winget (Windows 10+):
  winget install --id Git.Git -e --source winget

Option C — Chocolatey:
  choco install git

2.2 Linux
───────────────────────────────────────────────────────────────────────────────
Debian / Ubuntu / Mint:
  sudo apt update && sudo apt install git -y

Fedora / RHEL / CentOS:
  sudo dnf install git -y

Arch Linux:
  sudo pacman -S git

Alpine:
  apk add git

Verify: git --version

2.3 macOS
───────────────────────────────────────────────────────────────────────────────
Option A — Xcode Command Line Tools (recommended):
  xcode-select --install

Option B — Homebrew:
  brew install git

Option C — Official binary installer from https://git-scm.com/download/mac

Verify: git --version

================================================================================
3. CONFIGURATION
================================================================================

Git has three configuration levels:
  • System-wide:  git config --system        (file: [prefix]/etc/gitconfig)
  • Global:       git config --global        (file: ~/.gitconfig)
  • Local:        git config --local         (file: .git/config in the repo)

Priority: local > global > system

3.1 Identity (Required — affects commit authorship)

================================================================================
4. BRANCHING STRATEGIES
================================================================================

A branching strategy defines how teams create, merge, and manage branches.

4.1 Feature Branch Workflow
───────────────────────────────────────────────────────────────────────────────
Core idea: Every new feature or bugfix gets its own branch off main.

  # Start a feature
  git checkout -b feature/user-auth main

  # Work, commit, push
  git add -A && git commit -m "Add JWT authentication"
  git push -u origin feature/user-auth

  # When done, create a pull request to merge into main

Pros: Simple, works for small teams, CI-friendly.
Cons: No formal release management.

4.2 GitFlow
───────────────────────────────────────────────────────────────────────────────
Created by Vincent Driessen (2010). Uses two permanent branches + three supporting.

Permanent branches:
  • main       — Production-ready code. Every commit here is a release.
  • develop    — Integration branch for features.

Supporting branches:
  • feature/*  — Branched from develop, merged back to develop.
  • release/*  — Branched from develop, merged to main AND develop.
  • hotfix/*   — Branched from main, merged to main AND develop.

  # Start a release
  git checkout -b release/1.2.0 develop
  # Fix final bugs, bump version, then:
  git checkout main && git merge release/1.2.0 --no-ff
  git tag -a v1.2.0 -m "Release 1.2.0"
  git checkout develop && git merge release/1.2.0 --no-ff
  git branch -d release/1.2.0

Pros: Clear process, good for scheduled releases.
Cons: Complex, heavy overhead for continuous delivery teams.

4.3 GitHub Flow
───────────────────────────────────────────────────────────────────────────────
Simpler alternative: anything in main is deployable.

  1. Create a branch from main.
  2. Commit changes.
  3. Open a pull request.
  4. Discuss & review.
  5. Merge into main.
  6. Deploy immediately.

Pros: Dead simple, perfect for CI/CD.
Cons: No release branches — requires strong automated testing.

4.4 Trunk-Based Development
───────────────────────────────────────────────────────────────────────────────
All developers commit directly to main (or a short-lived branch < 1 day).

  • Feature flags hide incomplete work in production.
  • Very short-lived branches (hours, not days).
  • Frequent merges (multiple times daily).

Pros: Minimal merge hell, fast CI, ideal for DevOps.
Cons: Requires feature flags infrastructure, strong discipline.

4.5 General Branching Commands
───────────────────────────────────────────────────────────────────────────────
  git branch                        # List local branches (* = current)

================================================================================
5. MERGING vs REBASING
================================================================================

Both integrate changes from one branch into another — they differ in history.

5.1 Merging
───────────────────────────────────────────────────────────────────────────────
Creates a merge commit that ties two branch histories together.

  git checkout main
  git merge feature/login

Types:
  • Fast-forward merge: No merge commit if main hasn't diverged.
    git merge --ff-only feature/login  (fail if not fast-forwardable)

  • Three-way merge: Creates a merge commit.
    git merge --no-ff feature/login    (force merge commit always)

Pros: Preserves exact chronology, safe (non-destructive).
Cons: Can create messy history with many merge commits.

5.2 Rebasing
───────────────────────────────────────────────────────────────────────────────
Re-applies commits from your branch on top of another branch's tip, rewriting
history.

  git checkout feature/login
  git rebase main

What happens:
  1. Git finds the common ancestor between feature/login and main.
  2. All commits after that ancestor are saved as patches.
  3. The feature branch is reset to main's tip.
  4. Patches are re-applied one by one.

Pros: Clean linear history, easier to bisect.
Cons: Rewrites history — NEVER rebase shared/committed branches.
      Force push after rebase (git push --force-with-lease).

5.3 Interactive Rebase
───────────────────────────────────────────────────────────────────────────────
Edit commit history:

  git rebase -i HEAD~3   # Rebase last 3 commits

Available actions:
  pick     — Use the commit as-is
  reword   — Change commit message
  edit     — Stop to amend the commit
  squash   — Combine with previous commit
  fixup    — Like squash but discard message
  drop     — Remove the commit

5.4 When to Use What
───────────────────────────────────────────────────────────────────────────────
Use merge when:
  • Merging a public/shared branch.
  • You want to preserve the exact timeline.
  • You need an explicit integration event.

Use rebase when:
  • Cleaning up local commits before pushing.
  • Updating a feature branch with latest main.
  • You want a linear history.

Golden rule: Only rebase commits that exist only in your local repository.


================================================================================
7. STASHING
================================================================================

git stash temporarily shelves uncommitted changes so you can work on something
else, then re-apply them later.

7.1 Basic Commands
───────────────────────────────────────────────────────────────────────────────
  git stash                  # Stash tracked files (default: -m "WIP")
  git stash push -m "Fix WIP"  # Stash with a message
  git stash pop              # Apply latest stash and remove it from stack
  git stash apply            # Apply latest stash but keep it on stack
  git stash list             # List all stashes: stash@{0}, stash@{1}, ...
  git stash drop stash@{2}   # Delete a specific stash
  git stash clear            # Delete all stashes

7.2 Stashing Untracked/Ignored Files
───────────────────────────────────────────────────────────────────────────────
  git stash -u               # Include untracked files (--include-untracked)
  git stash -a               # Include all files including ignored (--all)

7.3 Partial Stash (Specific Files)
───────────────────────────────────────────────────────────────────────────────
  git stash push -m "config only" -- config.json

7.4 Create a Branch from a Stash
───────────────────────────────────────────────────────────────────────────────
  git stash branch <new-branch> stash@{0}

This creates a branch, checks it out, applies the stash, and drops it — useful
when you stashed on the wrong branch.

================================================================================
8. TAGGING
================================================================================

Tags mark specific commits as significant (releases, versions).

8.1 Types
───────────────────────────────────────────────────────────────────────────────
  • Lightweight: Just a pointer to a commit.
  • Annotated: Stored as full objects with tagger name, email, date, message,
    and optionally a GPG signature. Recommended for releases.

8.2 Commands
───────────────────────────────────────────────────────────────────────────────
  # Lightweight tag
  git tag v1.0.0

  # Annotated tag
  git tag -a v1.0.0 -m "Release version 1.0.0"

  # Tag a specific commit (not HEAD)
  git tag -a v0.9.0 9fceb02 -m "Beta release"

  # List tags
  git tag                       # Simple list

================================================================================
9. .gitignore PATTERNS
================================================================================

A .gitignore file tells Git which files to ignore (not track).

9.1 Pattern Syntax
───────────────────────────────────────────────────────────────────────────────
  # This is a comment (blank lines are ignored too)

  # File/directory by exact name
  secrets.json
  .env

  # Wildcard
  *.log
  *.tmp

  # Directory (trailing slash ignores everything inside it)
  node_modules/
  build/
  dist/
  .next/

  # Match anywhere in the tree
  .DS_Store

  # Match only at root (prefixed with /)
  /vendor
  /config/database.yml

  # Negate pattern (override previous ignore)
  !important.log

  # Recursive directory pattern
  logs/
  logs/**/debug.log

  # Character range
  photo[0-9].jpg

9.2 Common .gitignore Examples
───────────────────────────────────────────────────────────────────────────────
  # Node.js
  node_modules/
  npm-debug.log*
  .npm/
  .env

  # Python
  __pycache__/
  *.py[cod]
  *.egg-info/

================================================================================
11. PULL REQUESTS WORKFLOW
================================================================================

A Pull Request (PR) requests merging changes from one branch into another,
typically via GitHub, GitLab, or Bitbucket.

11.1 Standard Workflow
───────────────────────────────────────────────────────────────────────────────
  1. git checkout -b feature/awesome main
  2. git add -A && git commit -m "Implement awesome feature"
  3. git push -u origin feature/awesome
  4. Open a PR on the platform's web UI.
  5. Code review happens — push additional commits as needed.
  6. Merge the PR (options: merge commit, squash & merge, rebase & merge).
  7. Delete feature branch:
     git branch -d feature/awesome
     git push origin --delete feature/awesome

11.2 Best Practices
───────────────────────────────────────────────────────────────────────────────
  • Keep PRs small and focused.
  • Write descriptive titles and descriptions.
  • Reference issues: "Closes #42".
  • Keep branch up to date with base branch.
  • Run tests before opening/re-requesting review.
  • Use draft PRs for WIP.
  • Squash fixup commits before merging.

11.3 Fetch a PR Locally
───────────────────────────────────────────────────────────────────────────────
  git fetch origin pull/PR_ID/head:pr-branch-name
  git fetch origin pull/123/head:pr-123
  git checkout pr-123

================================================================================
12. git bisect
================================================================================

Binary search through commit history to find the commit that introduced a bug.
For N commits, takes only log2(N) steps.

12.1 Manual Bisect
───────────────────────────────────────────────────────────────────────────────
  git bisect start
  git bisect bad              # Mark HEAD as bad
  git bisect good v1.0.0      # Mark known good commit
  # Git checks out a midpoint. Test, then:
  git bisect good             # If this commit is good
  git bisect bad              # If this commit is bad
  git bisect reset

================================================================================
14. COMMON TROUBLESHOOTING
================================================================================

14.1 "Committed in the wrong branch"
───────────────────────────────────────────────────────────────────────────────
  git reset --soft HEAD~1
  git stash
  git checkout correct-branch
  git stash pop
  git add -A && git commit -m "..."

14.2 "Forgot to add a file"
───────────────────────────────────────────────────────────────────────────────
  git add forgotten-file.txt
  git commit --amend --no-edit

14.3 "Change last commit message"
───────────────────────────────────────────────────────────────────────────────
  git commit --amend -m "New message"

14.4 "Already pushed, need to fix"
───────────────────────────────────────────────────────────────────────────────
  git commit --amend
  git push --force-with-lease

14.5 "Merge conflict during pull"
───────────────────────────────────────────────────────────────────────────────
  git merge --abort
  git pull --rebase origin main

14.6 "Detached HEAD"
───────────────────────────────────────────────────────────────────────────────
  git switch -c new-branch-name
  # OR
  git switch main

14.7 "Accidentally committed sensitive data"
───────────────────────────────────────────────────────────────────────────────
  git rm --cached secrets.json
  echo "secrets.json" >> .gitignore
  git filter-branch --force --index-filter \
    "git rm --cached --ignore-unmatch secrets.json" \
    --prune-empty --tag-name-filter cat -- --all
  # Or use BFG Repo-Cleaner:
  java -jar bfg.jar --delete-files secrets.json
  git push --force --all
  # Rotate leaked credentials immediately!

14.8 "Changes lost after rebase"
───────────────────────────────────────────────────────────────────────────────
  git reflog
  git checkout -b recovery-branch <sha-before-rebase>

14.9 "Cannot push — remote diverged"
───────────────────────────────────────────────────────────────────────────────
  git fetch origin
  git rebase origin/main
  git push --force-with-lease

14.10 "Wrong upstream tracking"
───────────────────────────────────────────────────────────────────────────────
  git branch -u origin/main
  git push -u origin HEAD

14.11 "File permissions changed"
───────────────────────────────────────────────────────────────────────────────
  git config core.filemode false

14.12 "Large file committed"
───────────────────────────────────────────────────────────────────────────────
  git rev-list --objects --all | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | awk '/^blob/ {print $3, $4}' | sort -rn | head -10
  java -jar bfg.jar --strip-blobs-bigger-than 10M

14.13 "Modified but diff is empty"
───────────────────────────────────────────────────────────────────────────────
  git config core.autocrlf
  git add --renormalize .

================================================================================
END OF GIT DETAILED GUIDE
================================================================================


12.2 Automated Bisect
───────────────────────────────────────────────────────────────────────────────
  git bisect start
  git bisect bad HEAD
  git bisect good v1.0.0
  git bisect run npm test     # Exits 0=good, non-zero=bad

12.3 Skipping & Viewing
───────────────────────────────────────────────────────────────────────────────
  git bisect skip
  git bisect log
  git bisect visualize

================================================================================
13. git reflog
================================================================================

Records local operations that move branch pointers and HEAD. Safety net for
recovering "lost" commits. Only exists locally (never pushed).

Expiry: 90 days (reachable), 30 days (unreachable).

13.1 Viewing
───────────────────────────────────────────────────────────────────────────────
  git reflog                          # Short format
  git reflog --relative-date
  git reflog main                     # Specific branch
  git reflog --all

13.2 Recovery Scenarios
───────────────────────────────────────────────────────────────────────────────
  A) After git reset --hard:
     git reflog && git reset --hard HEAD@{1}

  B) Deleted a branch:
     git reflog && git branch recovered-branch <sha>

  C) Rebase went wrong:
     git reflog && git reset --hard HEAD@{3}

  D) Cherry-pick lost commit:
     git cherry-pick <sha>

  .venv/
  venv/
  .pytest_cache/

  # Rust
  target/
  Cargo.lock

  # Go
  *.exe
  *.exe~
  *.dll
  *.so
  *.dylib
  vendor/

  # Java
  *.class
  *.jar
  *.war
  target/
  .gradle/
  build/

  # IDE / Editor
  .vscode/
  .idea/
  *.swp
  *.swo
  *~

  # OS
  .DS_Store
  Thumbs.db

9.3 Global .gitignore
───────────────────────────────────────────────────────────────────────────────
  git config --global core.excludesFile ~/.gitignore_global

================================================================================
10. REMOTE MANAGEMENT
================================================================================

10.1 What is a Remote?
───────────────────────────────────────────────────────────────────────────────
A remote is a URL pointing to another copy of the repo.
  • origin    — Your fork or primary remote.
  • upstream  — The original repo (fork workflows).

10.2 Commands
───────────────────────────────────────────────────────────────────────────────
  git remote -v                              # List remotes
  git remote add origin <url>                # Add a remote
  git remote rename origin upstream          # Rename
  git remote remove old-remote               # Remove
  git remote set-url origin <url>            # Change URL
  git remote show origin                     # Show details
  git fetch origin                           # Fetch objects
  git fetch --prune origin                   # Fetch & prune
  git pull origin main                       # Pull (fetch+merge)
  git push origin main
  git push -u origin feature/branch          # Push + set upstream

10.3 Fork Workflow (Upstream)
───────────────────────────────────────────────────────────────────────────────
  1. Fork upstream repo on GitHub.
  2. Clone: git clone https://github.com/YOU/repo.git
  3. Add upstream: git remote add upstream <original-url>
  4. Sync: git fetch upstream
            git checkout main
            git merge upstream/main
  5. Push:  git push origin main

  git tag -l "v1.*"             # Filter by pattern
  git tag -n                    # List with messages

  # Show tag details
  git show v1.0.0

  # Delete a tag
  git tag -d v1.0.0             # Local
  git push origin --delete v1.0.0  # Remote

  # Push tags
  git push origin v1.0.0        # Single tag
  git push origin --tags        # All tags

8.3 Semantic Versioning Convention
───────────────────────────────────────────────────────────────────────────────
  vMAJOR.MINOR.PATCH

  • MAJOR: Breaking changes.
  • MINOR: New features, backward-compatible.
  • PATCH: Bug fixes, backward-compatible.

  Example: v2.4.1

================================================================================
6. RESOLVING MERGE CONFLICTS
================================================================================

A conflict occurs when two branches modify the same lines of a file differently.

6.1 Conflict Markers
───────────────────────────────────────────────────────────────────────────────
When a conflict happens, Git marks the file:

  <<<<<<< HEAD
  This is the current branch's version.
  =======
  This is the incoming branch's version.
  >>>>>>> feature/other

Parts:
  • <<<<<<< HEAD — Start of our changes.
  • =======      — Separator between our and their changes.
  • >>>>>>> ref  — End of their changes, shows branch name/hash.

6.2 Resolution Steps
───────────────────────────────────────────────────────────────────────────────
  1. Identify conflicted files:
     git status   # Lists "both modified" files

  2. Open each conflicted file and edit to keep the desired code.
     Remove the conflict markers (<<<<<<<, =======, >>>>>>>).

  3. After resolving all conflicts in a file:
     git add <file>

  4. Complete the merge:
     git commit   # Git provides a pre-filled merge commit message

6.3 Tools
───────────────────────────────────────────────────────────────────────────────
  git mergetool   # Opens configured merge tool (vimdiff, VS Code, etc.)

  # Configure VS Code as merge tool:
  git config --global merge.tool vscode
  git config --global mergetool.vscode.cmd "code --wait \$MERGED"

  # Use ours/theirs strategy for bulk resolution:
  git checkout --ours -- <file>     # Keep current branch version
  git checkout --theirs -- <file>   # Keep incoming branch version

6.4 Aborting a Merge
───────────────────────────────────────────────────────────────────────────────
  git merge --abort    # Go back to pre-merge state

  git branch -a                     # List all branches (local + remote)
  git branch -d <name>              # Delete a merged branch
  git branch -D <name>              # Force delete unmerged branch
  git push origin --delete <name>   # Delete a remote branch
  git branch -m <old> <new>         # Rename a branch
  git checkout -b <new> <base>      # Create & switch from base
  git switch -c <new>               # Modern alternative (Git 2.23+)

───────────────────────────────────────────────────────────────────────────────
  git config --global user.name "Your Name"
  git config --global user.email "your.email@example.com"

Verify:
  git config --global --list

3.2 Recommended Global Settings
───────────────────────────────────────────────────────────────────────────────
  git config --global init.defaultBranch main
  git config --global pull.rebase false     # Use merge on pull (default)
  git config --global fetch.prune true      # Auto-prune stale remote-tracking refs
  git config --global diff.colorMoved zebra # Color moved code blocks
  git config --global core.editor "code --wait"  # VS Code as editor
  git config --global core.autocrlf input   # For macOS/Linux
  # Windows users:
  git config --global core.autocrlf true

3.3 Aliases (Time-savers)
───────────────────────────────────────────────────────────────────────────────
  git config --global alias.st status
  git config --global alias.co checkout
  git config --global alias.br branch
  git config --global alias.ci commit
  git config --global alias.df diff
  git config --global alias.dfc "diff --cached"
  git config --global alias.lg "log --oneline --graph --all --decorate"
  git config --global alias.undo "reset --soft HEAD~1"
  git config --global alias.amend "commit --amend --no-edit"

  • Staging Area (Index): A middle layer between working tree and repository,
    where you prepare changes before committing.
  • Remote: A copy of the repository hosted elsewhere (GitHub, GitLab, Bitbucket).

Git tracks content by content-addressable storage — every file and directory is
checksummed, and Git retrieves data by its hash. This guarantees data integrity.

