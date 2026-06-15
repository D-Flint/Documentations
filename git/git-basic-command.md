Git Basic Commands
===================

1. git init
   - Initializes a new Git repository in the current directory.
   - Creates a hidden .git folder that tracks all changes.

2. git clone <repository-url>
   - Creates a local copy of a remote repository.
   - Example: git clone https://github.com/user/repo.git

3. git status
   - Shows the current state of the working directory.
   - Displays modified, staged, and untracked files.

4. git add <file>
   - Stages a file for commit.
   - Use git add . to stage all changes in the directory.

5. git commit -m "message"
   - Saves staged changes with a descriptive message.
   - Example: git commit -m "Fix login bug"

6. git push
   - Uploads local commits to a remote repository.
   - Usually used after committing changes.
   - Ex: git push -u origin main

7. git pull
   - Fetches and merges changes from a remote repository.
   - Keeps your local branch up to date.

8. git branch
   - Lists all branches in the repository.
   - Use git branch <name> to create a new branch.

9. git checkout <branch>
   - Switches to a different branch.
   - Use git checkout -b <branch> to create and switch.

10. git merge <branch>
    - Merges changes from another branch into the current one.
    - Example: git merge feature-branch

11. git log
    - Shows the commit history of the repository.
    - Use git log --oneline for a compact view.

12. git diff
    - Displays differences between files or commits.
    - Use git diff --staged to see staged changes.
