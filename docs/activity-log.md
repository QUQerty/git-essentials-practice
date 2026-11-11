# Lecture 1 activity log

Commands run against this repository, with the behaviour observed. Terminal
output is reproduced as it was printed.

## Activity 1 - Create and initialize a repository

```
mkdir git-essentials-practice && cd git-essentials-practice
git init -b main
git add . && git commit -m "Initialize repository with README and notes"
gh repo create QUQerty/git-essentials-practice --public --source=. --remote=origin
git push -u origin main
```

`git remote -v` confirmed the association and `git status -sb` showed
`## main...origin/main`, meaning local `main` now tracks the remote.

## Activity 2 - Clone and associate a repository

```
git clone https://github.com/QUQerty/git-essentials-practice.git git-essentials-clone
cd git-essentials-clone
git remote -v          # origin -> the same repository
git status -sb         # ## main...origin/main (tracking set up by clone)
git pull --rebase      # Already up to date.
echo "Line B..." >> notes.txt
git add notes.txt && git commit -m "Add a line from the cloned copy"
git push origin main
```

A clone sets up `origin` and the upstream tracking branch automatically, so none
of Activity 1's remote wiring has to be repeated.

## Activity 3 - Branching and merging

```
git checkout -b feature-greeting     # Switched to a new branch 'feature-greeting'
git branch --show-current            # feature-greeting
git add greeting.txt && git commit -m "Add greeting file on feature branch"
git checkout main
git merge feature-greeting           # 1 file changed, 1 insertion(+)
git branch -d feature-greeting       # Deleted branch feature-greeting (was 5db52fd)
```

Because `main` had not moved on, this was a fast-forward merge: no merge commit
was created, `main` simply advanced to the feature branch's commit.

## Activity 4 - Revert, amend, reset

### Amend (rewrites the last commit)

| Scenario | Command | Result |
|---|---|---|
| Fix a message typo | `git commit --amend -m "Add third line to notes"` | `c0522b1` -> `2aefcba` |
| Add a forgotten file | `git add scratch.txt && git commit --amend --no-edit` | commit now holds 2 files |
| Remove a file | `git rm --cached scratch.txt && git commit --amend --no-edit` | back to 1 file |

The commit hash changed every time. Amend does not edit a commit, it replaces
it - which is why it is only safe before pushing.

### Revert (safe for shared history)

```
git revert --no-edit HEAD
```

Produced a new commit `1d1dd2a Revert "Add third line to notes"`. The original
`df2be56` is still in the log. Nothing was rewritten, so this is the safe way to
undo a commit that others may already have pulled.

### Reset (three modes, run on a throwaway branch)

Starting from three demo commits:

```
git reset --soft HEAD~1    # commit undone, changes still STAGED    -> "M  reset-demo.txt"
git reset --mixed HEAD     # changes kept but UNSTAGED              -> " M reset-demo.txt"
git reset --hard f6a664f   # pointer moved, all changes DISCARDED   -> working tree clean
```

The staged/unstaged column in `git status --short` is what distinguishes soft
from mixed. Hard reset discarded the file content permanently. This was done on
a branch that was deleted afterwards, because reset rewrites history and `main`
had already been pushed.

## Activity 5 - Rebase and resolving conflicts

`main` and `feature-rebase` both edited the same line of `notes.txt`, so the
rebase stopped with a conflict:

```
Could not apply 35ec3fb... Update line A on the feature branch
```

```
Git Essentials practice file.
<<<<<<< HEAD
Line A: changed on main.
=======
Line A: changed on the feature branch.
>>>>>>> 35ec3fb (Update line A on the feature branch)
Line B: added from the cloned copy.
```

`git status --short` reported `UU notes.txt` (both sides modified).

**Abort first**, to show the escape hatch:

```
git rebase --abort     # returns to 35ec3fb, markers gone, branch untouched
```

**Then resolve properly** by keeping both lines:

```
git add notes.txt
git rebase --continue  # Successfully rebased and updated refs/heads/feature-rebase.
```

The rebased commit came out as `66ec705`, not `35ec3fb` - rebase reapplies
commits, creating new IDs. The resulting history is linear with no merge commit,
which is the practical difference from Activity 3's merge.
