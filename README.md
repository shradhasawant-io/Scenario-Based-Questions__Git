# Scenario-Based Questions: Git

In real software companies, knowing `git add`, `git commit`, `git push`, and `git pull` is not enough. Developers also need to know how to recover from mistakes, manage commit history, transfer specific changes between branches, and maintain clean Git history.

Let's explore advanced Git commands through practical workplace situations, especially from a Java/Spring Boot developer's perspective.

### Topics covered:

1. `git rebase` –>  Reapply commits on top of another branch.
2. `git rebase -i` –>  Edit, squash, reorder, or remove commits.
3. `git reset` –>  Move the current branch pointer backward.
4. `git cherry-pick` –>  Apply a specific commit to another branch.
5. `git revert` –>  Safely undo an existing commit.
6. `git reflog` –>  Recover lost commits and branch references.
7. `git stash` –>  Temporarily save unfinished work.
8. `git commit --amend` –>  Modify the latest commit.
9. `git bisect` –>  Find the commit that introduced a bug.
10. `git restore` –>  Restore files or discard unwanted changes.

## Scenario 1
Your feature branch is behind `develop`, and your senior asks you to rebase it

**Command:** `git rebase`

### Workplace situation

You are implementing a Customer API in:

feature/customer-api

Meanwhile, other developers have merged their work into `develop`.

Your senior says:

Before raising your PR, rebase your feature branch with the latest develop branch.

### Why does this happen?

Your feature branch was created several days ago. Since then, `develop` has received multiple commits.

Your team wants your feature branch to include those latest changes and maintain a cleaner, linear commit history.

### Step-by-step solution

Step 1. Check your current branch

git branch

Step 2. Fetch the latest remote changes

git fetch origin

Step 3. Switch to your feature branch

git switch feature/customer-api

Step 4. Rebase your branch onto develop

git rebase origin/develop

Git takes your feature commits and reapplies them on top of the latest `origin/develop`.

Before:

\\

A---B---C  develop

\\

D---E  feature/customer-api

After:

A---B---C---D'---E'  feature/customer-api

Notice that `D` and `E` become new commits (`D'` and `E'`).

Step 5. If conflicts occur

Open the conflicting files, understand both implementations, resolve the conflicts, and then:

git add .

git rebase --continue

If you want to cancel the entire rebase:

git rebase --abort

Step 6. Run your tests

For your Spring Boot project:

```

mvn clean test

```

Step 7. Push your updated feature branch

If this is your own feature branch and it was already pushed before rebasing:

```

git push --force-with-lease origin feature/customer-api

```

Why? Rebase creates new commit IDs, so the remote branch may no longer accept a normal fast-forward push.

### Impact

- Local: Feature branch history is rewritten.

- Remote: Unchanged until you push.

- GitHub: Existing PR gets updated after the push.

### Common mistakes

- Rebasing the wrong branch.

- Forgetting to fetch the latest `develop`.

- Using `git push --force` instead of considering `--force-with-lease`.

- Rebasing a shared branch without consulting teammates.

Developer perspective: Rebase is particularly useful for keeping your personal feature branch up to date. Avoid rewriting history that other developers are already using.

Reference: [Official Git rebase documentation](https://git-scm.com/docs/git-rebase?utm_source=chatgpt.com).

## Scenario 2: Your PR has 10 unnecessary commits, and your senior asks you to clean up the history

Command: `git rebase -i`

### Workplace situation

You are implementing an Account Management API.

During development, you created these commits:

```

a1 - Implement account entity

a2 - Add account repository

a3 - Add account service

a4 - Fix service error

a5 - Fix service error again

a6 - Update controller

a7 - Fix controller validation

a8 - Update API response

```

Your senior reviews your PR and says:

> Please squash unnecessary commits. Keep the PR history clean and meaningful.

### Why does this happen?

During development, developers often create small intermediate commits.

These are useful for saving progress, but they can make the final PR history unnecessarily noisy.

Interactive rebase lets you combine, reorder, edit, or remove commits.

### Step-by-step solution

Step 1. Check your commit history

```

git log --oneline

```

Step 2. Start interactive rebase

Suppose you want to review the last eight commits:

```

git rebase -i HEAD~8

```

Your editor opens with something similar to:

```

pick a1 Implement account entity

pick a2 Add account repository

pick a3 Add account service

pick a4 Fix service error

pick a5 Fix service error again

pick a6 Update controller

pick a7 Fix controller validation

pick a8 Update API response

```

Step 3. Change the instructions

For example:

```

pick a1 Implement account entity

fixup a2 Add account repository

fixup a3 Add account service

fixup a4 Fix service error

fixup a5 Fix service error again

fixup a6 Update controller

fixup a7 Fix controller validation

fixup a8 Update API response

```

Save and close the editor.

Git combines the changes into one commit, keeping the first commit message.

### Understand the interactive rebase options

| Option   | Meaning                                       |
| -------- | ------------------------------------------------- |
| `pick`   | Keep the commit                                   |
| `reword` | Change the commit message                     |
| `edit`   | Stop and modify the commit                        |
| `squash` | Combine commits and edit combined message     |
| `fixup`  | Combine commits and discard later commit messages |
| `drop`   | Remove a commit                                   |

Step 4. Verify the new history

```

git log --oneline

```

Expected:

```

z123 Implement account management API

```

Step 5. Update GitHub

If these commits were already pushed:

```

git push --force-with-lease origin feature/account-management

```

### Impact

- Local: Multiple commits become one.

- Remote: Old branch history remains until you push.

- GitHub: The PR reflects the rewritten commit history.

### Common mistakes

- Squashing commits that are already shared with other developers.

- Accidentally using `drop` on a required commit.

- Reordering dependent commits incorrectly.

- Forgetting to test after rewriting history.

Developer perspective: Interactive rebase is useful before submitting a PR when your team prefers a clean commit history.

Reference: [Interactive rebase instructions](https://git-scm.com/docs/git-rebase.html?utm_source=chatgpt.com).

## Scenario 3: You made three incorrect commits and want to undo them but keep your code changes

Command: `git reset --soft`

### Workplace situation

You are implementing the Deposit module.

You accidentally created three commits:

```

Commit 1: Add deposit entity

Commit 2: Add deposit service

Commit 3: Add deposit controller

```

Later, you realize these commits should have been combined into one meaningful commit.

You want to remove the three commits from local history without losing your code.

### Why does this happen?

Sometimes developers commit too frequently or create commits with incorrect messages.

If the commits are local and unpublished, `git reset` can help reorganize them.

### Step-by-step solution

Step 1. Check the latest commits

```

git log --oneline -5

```

Step 2. Reset the branch by three commits

```

git reset --soft HEAD~3

```

What happens?

- The last three commits are removed from the current branch history.

- All changes introduced by those commits remain.

- The changes are placed in the staging area.

Step 3. Verify the changes

```

git status

```

Step 4. Review staged changes

```

git diff --cached

```

Step 5. Create one meaningful commit

```

git commit -m "feat: implement deposit management module"

```

Step 6. Verify the history

```

git log --oneline -5

```

### Impact

- Local: Three commits are replaced by one new commit.

- Files: Your code changes are retained.

- Remote: No effect until you push.

### Important: Understand all three reset modes

This is one of the most important Git concepts.

| Command                    | Commit history | Staging area     | Working files                    |
| -------------------------- | -------------- | ---------------- | -------------------------------- |
| `git reset --soft HEAD~1`  | Moves backward | Changes retained | Changes retained             |
| `git reset --mixed HEAD~1` | Moves backward | Changes unstaged | Changes retained                 |
| `git reset --hard HEAD~1`  | Moves backward | Reset            | Reset; tracked changes discarded |

### When should you use each?

- `--soft`: You want to recommit changes.

- `--mixed`: You want to reorganize and selectively stage changes.

- `--hard`: You intentionally want to discard changes and return to an earlier state.

### Important warning

Never casually run:

```

git reset --hard HEAD~3

```

It can discard uncommitted changes and remove the commits from the current branch reference. Recovery may be possible through reflog, but you should not rely on it.

Developer perspective: Use reset primarily to clean up local, unpublished history. For already shared commits, prefer `git revert`.

Reference: [Official Git reset documentation](https://git-scm.com/docs/git-reset.html?utm_source=chatgpt.com).

## Scenario 4: A critical production bug is fixed in another branch, and you need only that one fix

Command: `git cherry-pick`

### Workplace situation

Your team maintains two branches:

```

main

develop

```

A developer fixes a critical login bug in:

```

hotfix/login-bug

```

The fix is committed, but your release branch also needs that specific change.

You do not want to merge the entire hotfix branch because it contains other changes that are not ready for release.

### Why does this happen?

Sometimes a particular commit is needed in another branch, but merging the entire branch would introduce unwanted changes.

`git cherry-pick` lets you apply the changes from a specific commit.

### Step-by-step solution

Step 1. Identify the required commit

```

git log --oneline hotfix/login-bug

```

Example:

```

a123abc Fix login authentication failure

b456def Refactor authentication service

c789ghi Update login documentation

```

You need only `a123abc`.

Step 2. Switch to your target branch

```

git switch release/1.2

```

Step 3. Apply the selected commit

```

git cherry-pick a123abc

```

Git applies the changes introduced by that commit and creates a new commit on the current branch.

Step 4. Verify the changes

```

git log --oneline -5

```

Step 5. Test your application

```

mvn clean test

```

Step 6. Push your target branch

```

git push origin release/1.2

```

### Impact

- Source branch: `hotfix/login-bug` remains unchanged.

- Target branch: Receives the selected changes as a new commit.

- Remote: Updated only after pushing.

### Additional useful cherry-pick commands

Apply multiple specific commits:

```

git cherry-pick a123abc b456def

```

Apply changes without creating a commit immediately:

```

git cherry-pick -n a123abc

```

Abort a cherry-pick if it encounters conflicts and you decide not to proceed:

```

git cherry-pick --abort

```

### Common mistakes

- Cherry-picking the wrong commit.

- Forgetting that the target commit gets a new commit ID.

- Cherry-picking dependent changes without their required dependencies.

- Cherry-picking a commit that has already been integrated.

Developer perspective: Cherry-pick is useful for hotfixes, backporting bug fixes to release branches, and selectively transferring changes between branches.

Reference: [Official Git cherry-pick documentation](https://git-scm.com/docs/git-cherry-pick.html?utm_source=chatgpt.com).

## Scenario 5: You cherry-picked a commit, but Git reports a conflict

Command: `git cherry-pick --continue`

### Workplace situation

Your teammate implemented customer validation in:

```

feature/customer-validation

```

You need the same validation change in:

```

release/1.2

```

You execute:

```

git cherry-pick a123abc

```

But Git reports:

```

error: could not apply a123abc

CONFLICT (content): CustomerService.java

```

### Why does this happen?

The target branch has a different version of `CustomerService.java`.

Git cannot automatically apply the selected commit because the relevant code has changed.

### Step-by-step solution

Step 1. Check the current operation

```

git status

```

Step 2. Open the conflicting file

You may see:

```

<<<<<<< HEAD

public void validateCustomer(Customer customer) {

// Existing implementation

}

\=======

public void validateCustomer(Customer customer) {

// New validation implementation

}

>>>>>>> a123abc

```

Step 3. Resolve the conflict

Understand both implementations and produce the correct final code.

Remove the conflict markers.

Step 4. Stage the resolved file

```

git add CustomerService.java

```

Step 5. Continue cherry-picking

```

git cherry-pick --continue

```

Step 6. Verify the commit

```

git log --oneline -5

```

Step 7. Test and push

```

mvn clean test

git push origin release/1.2

```

What if you want to cancel the cherry-pick?

```

git cherry-pick --abort

```

### Important distinction

| Command                  | Purpose                         |
| ---------------------------- | ----------------------------------- |
| `git rebase --continue`  | Continue an interrupted rebase  |
| `git cherry-pick --continue` | Continue an interrupted cherry-pick |
| `git merge --continue`   | Complete an interrupted merge   |
| `git cherry-pick --abort` | Cancel the cherry-pick operation |

Developer perspective: Cherry-picking is not always a simple copy-paste operation. The same change may need adjustments because different branches may have different code structures.

Reference: [Cherry-pick conflict handling](https://git-scm.com/docs/git-cherry-pick?c=15OneBitJRx&utm_source=chatgpt.com).

## Scenario 6: You pushed a wrong commit to GitHub, and your senior asks you to undo it safely

Command: `git revert`

### Workplace situation

You implemented a new withdrawal API and pushed it to:

```

feature/withdrawal-api

```

But after code review, your senior discovers that the implementation has a serious business logic issue.

Your commit is already pushed, and another developer has pulled the branch.

Your senior says:

> Undo this commit without rewriting shared history.

### Why does this happen?

Once a commit is pushed and shared, other developers may depend on it.

Using `git reset` and force-pushing can disrupt their work.

`git revert` is generally the safer option because it creates a new commit that reverses an earlier commit.

### Step-by-step solution

Step 1. Find the incorrect commit

```

git log --oneline

```

Example:

```

a123abc Implement withdrawal API

b456def Add account balance validation

c789ghi Add transaction entity

```

Step 2. Revert the incorrect commit

```

git revert a123abc

```

Git creates a new commit that reverses the changes introduced by `a123abc`.

Step 3. Verify the commit history

```

git log --oneline -5

```

Example:

```

d111aaa Revert "Implement withdrawal API"

a123abc Implement withdrawal API

b456def Add account balance validation

c789ghi Add transaction entity

```

Notice that the original commit still exists.

Step 4. Push the new commit

```

git push origin feature/withdrawal-api

```

### Impact

- Local: A new reversing commit is created.

- Remote: Updated after pushing.

- Other developers: Their existing commit history remains intact.

What if you want to revert without immediately creating a commit?

```

git revert --no-commit a123abc

```

You can inspect the resulting changes and create your own commit afterward.

### Common mistakes

- Using `reset --hard` on shared branches.

- Deleting the original commit's history.

- Reverting the wrong commit.

- Assuming reverting a commit deletes it from Git history.

Developer perspective: Remember this simple rule:

- Unpublished incorrect commit → Consider `reset`.

- Published incorrect commit → Prefer `revert`.

Reference: [Git documentation on reverting published changes](https://git-scm.com/docs/user-manual.html?utm_source=chatgpt.com).

## Scenario 7: You accidentally ran `git reset --hard` and lost your commits

Command: `git reflog`

### Workplace situation

You are working on the Reports module.

You created two important commits:

```

a111 Implement report service

b222 Add report filtering

```

You accidentally execute:

```

git reset --hard HEAD~2

```

Now both commits have disappeared from your current branch history.

You think your work is permanently lost.

### Why does this happen?

`git reset --hard` moves the current branch backward and resets the staging area and working directory.

However, Git maintains a local reference history called the reflog, which records movements of `HEAD` and branch references.

You may be able to recover the previous commit.

### Step-by-step solution

Step 1. Check the current history

```

git log --oneline

```

The two commits are no longer visible in the current branch's normal history.

Step 2. Check the reflog

```

git reflog

```

Example:

```

c333 HEAD@{0}: reset: moving to HEAD~2

b222 HEAD@{1}: commit: Add report filtering

a111 HEAD@{2}: commit: Implement report service

```

Step 3. Identify the commit you want to recover

Suppose `b222` is the latest commit you want to restore.

Step 4. Create a recovery branch first

```

git branch recovery/reports b222

```

This creates a new branch pointing to the recovered commit without changing your current branch.

Step 5. Inspect the recovered history

```

git log --oneline recovery/reports

```

Step 6. Switch to the recovery branch

```

git switch recovery/reports

```

Your previous committed work is now accessible again.

If you want to restore your original branch instead, verify the recovery branch first, then move the original branch pointer to the recovered commit:

```

git switch feature/reports

git reset --hard b222

```

Use carefully

Only perform the hard reset after confirming that you want to replace the current branch state and have protected any other work.

### Impact

- Reflog: Read from your local repository.

- Recovery branch: Created locally.

- Remote: Unchanged until you push.

### Common mistakes

- Assuming `git log` shows every recoverable commit.

- Running another reset without inspecting reflog.

- Choosing the wrong reflog entry.

- Waiting too long: reflog entries are subject to expiration and garbage collection.

Developer perspective: Reflog is one of the most useful Git recovery tools. Learn it before you ever need it in a production project.

Reference: [GitHub guide: How to undo almost anything with Git](https://github.blog/open-source/git/how-to-undo-almost-anything-with-git/?utm_source=chatgpt.com).

## Scenario 8: You forgot to include a file in your latest commit, and your senior has not reviewed your PR yet

Command: `git commit --amend`

### Workplace situation

You implemented a Customer API and committed:

```

git add CustomerController.java CustomerService.java

git commit -m "feat: implement customer API"

```

After committing, you realize you forgot to include:

```

CustomerRepository.java

```

You haven't yet pushed your latest commit.

### Why does this happen?

Developers sometimes forget to stage a file or make a small correction immediately after committing.

Instead of creating an unnecessary second commit, you can amend the latest commit.

### Step-by-step solution

Step 1. Check your working directory

```

git status

```

You should see `CustomerRepository.java` as modified or untracked.

Step 2. Stage the missing file

```

git add CustomerRepository.java

```

Step 3. Amend your latest commit

```

git commit --amend

```

Your editor opens with the previous commit message.

Keep or update it:

```

feat: implement customer API

```

Save and close.

Step 4. Verify the commit

```

git show --stat HEAD

```

This displays the files included in the latest commit.

Alternative: Amend without changing the commit message

```

git commit --amend --no-edit

```

What if you already pushed the commit?

Amending creates a new commit ID. If it is your own branch and nobody else depends on it, you may update the remote using:

```

git push --force-with-lease origin feature/customer-api

```

But coordinate first if the branch is shared.

### Impact

- Local: Previous latest commit is replaced with a new commit.

- Remote: No impact until pushing.

- GitHub PR: Updated after pushing the amended history.

### Common mistakes

- Amending the wrong commit.

- Forgetting that amend changes the commit ID.

- Amending shared commits without coordination.

- Accidentally including unrelated staged files.

Developer perspective: Use amend for correcting the most recent unpublished commit, not for casually modifying commits already shared with the team.

Reference: [Git documentation on commit amendment](https://github.com/git/git/blob/master/Documentation/user-manual.adoc?utm_source=chatgpt.com).

## Scenario 9: A bug appeared after several commits, and you don't know which commit introduced it

Command: `git bisect`

### Workplace situation

Your team has developed the Account Management module over several days.

Initially, everything worked correctly.

But after several new commits, the account balance API starts returning incorrect values.

You know:

- Commit `a111` had working functionality.

- Current commit `z999` has the bug.

- Somewhere between these commits, a developer introduced the problem.

Your senior asks:

> Identify the exact commit that introduced this issue.

### Why does this happen?

In a large project, dozens of commits may have been made since the last known working version.

Checking every commit manually is time-consuming.

Git bisect uses binary search to identify the problematic commit.

### Step-by-step solution

Step 1. Start bisect

```

git bisect start

```

Step 2. Mark the current version as bad

```

git bisect bad

```

Step 3. Mark the older working commit as good

```

git bisect good a111

```

Git selects an intermediate commit.

Step 4. Test the selected version

At each selected commit:

```

mvn clean test

```

Or execute your specific failing integration test.

If the bug exists:

```

git bisect bad

```

If the bug does not exist:

```

git bisect good

```

Step 5. Repeat

Git selects another commit based on your previous answer.

Continue until Git identifies the first bad commit.

Example output:

```

b456def is the first bad commit

```

Step 6. Inspect the problematic commit

```

git show b456def

```

You can also inspect the files changed:

```

git show --stat b456def

```

Now you can identify which code change introduced the issue.

Step 7. Finish bisect

```

git bisect reset

```

This returns you to the branch or commit you were on before starting bisect.

### Impact

- Git checks out historical commits during the investigation.

- Your existing branch history is not rewritten.

- Remote repository remains unchanged.

- `git bisect reset` returns you to your previous checkout.

### Common mistakes

- Marking a commit as good without testing it.

- Using inconsistent test conditions.

- Forgetting to run `git bisect reset`.

- Ignoring database migrations or external dependencies that make old versions difficult to test.

Developer perspective: Bisect is particularly useful when debugging regressions in large codebases where many developers contribute daily.

Reference: [Official Git bisect documentation](https://git-scm.com/docs/git-bisect?utm_source=chatgpt.com).

## Scenario 10: You are working on a feature, but your manager assigns an urgent task, and you need to temporarily save your changes

Commands: `git stash`, `git stash apply`, `git stash pop`

### Workplace situation

You are implementing:

```

feature/customer-report

```

You have unfinished changes in:

```

ReportController.java

ReportService.java

ReportRepository.java

```

Suddenly, your manager assigns you an urgent production bug.

You need to switch to:

```

hotfix/report-calculation

```

But your existing feature changes are incomplete and cannot be committed.

### Why does this happen?

Developers frequently need to switch between tasks.

Stash temporarily stores local changes so you can work on another branch without making an incomplete commit.

### Step-by-step solution

Step 1. Check your current changes

```

git status

```

Step 2. Save your unfinished work

```

git stash push -m "WIP: customer report feature"

```

If you also want to save untracked files:

```

git stash push -u -m "WIP: customer report feature"

```

Step 3. Switch to the hotfix branch

```

git switch develop

git pull --ff-only origin develop

git switch -c hotfix/report-calculation

```

Step 4. Fix the urgent bug

Make the necessary changes and test them.

Step 5. Commit the hotfix

```

git add .

git commit -m "fix: correct report calculation"

```

Step 6. Push the hotfix

```

git push -u origin hotfix/report-calculation

```

Step 7. Return to your feature branch

```

git switch feature/customer-report

```

Step 8. Check saved stashes

```

git stash list

```

Example:

```

stash@{0}: On feature/customer-report: WIP: customer report feature

```

Step 9. Restore your work

```

git stash pop

```

This applies the latest stash and removes it from the stash list if successful.

Alternatively:

```

git stash apply

```

This restores the changes but retains the stash entry.

### Additional commands

See saved changes:

```

git stash show -p stash@{0}

```

Remove a particular stash:

```

git stash drop stash@{0}

```

### Impact

- Stash: Stored locally, not automatically uploaded to GitHub.

- Hotfix: Committed and pushed independently.

- Original feature: Restored when you return to it.

### Common mistakes

- Forgetting to include untracked files when needed.

- Applying a stash to the wrong branch.

- Assuming stash is a permanent backup.

- Deleting a stash before verifying restored changes.

Developer perspective: Stash is a temporary workspace-management tool. It is not a replacement for commits or remote backups.

Reference: [Git user manual: Temporarily setting aside work](https://github.com/git/git/blob/master/Documentation/user-manual.adoc?utm_source=chatgpt.com).

### Quick revision: Which command should you use in which situation?

This is probably the most important table to remember for workplace scenarios and interviews.

| Workplace situation                | Git command      |
| -------------------------------------- | -------------------- |
| Feature branch is behind develop   | `git rebase`     |
| Need to clean up multiple commits  | `git rebase -i`  |
| Undo commits but retain staged changes | `git reset --soft`   |
| Undo commits and unstage changes   | `git reset --mixed`  |
| Discard local changes and commits  | `git reset --hard`   |
| Apply one commit to another branch | `git cherry-pick` |
| Undo an already-pushed commit      | `git revert`     |
| Recover accidentally lost commits  | `git reflog`     |
| Add forgotten files to latest commit   | `git commit --amend` |
| Identify which commit introduced a bug | `git bisect`     |
| Temporarily save unfinished changes | `git stash`      |

### Important differences: Rebase vs Cherry-Pick vs Merge

Developers frequently confuse these three commands.

| Feature                      | Rebase                                | Cherry-pick                      | Merge                    |
| -------------------------------- | ----------------------------------------- | ------------------------------------ | ---------------------------- |
| Purpose                      | Reapply your branch commits on a new base | Apply selected commits           | Integrate branch histories   |
| Works with                   | A series of commits                   | Specific commits                 | Entire branch histories  |
| Creates new commit IDs       | Yes, for reapplied commits            | Yes                              | Depends on merge strategy |
| Rewrites existing branch history | Yes                                   | No, source history remains unchanged | No                       |
| Common use                   | Update feature branch                 | Backport hotfix                  | Integrate completed features |

Remember:

- Rebase: Move your work onto a newer base.

- Cherry-pick: Take a particular change from somewhere else.

- Merge: Bring branch histories together.

[image](https://www.google.com/s2/favicons?domain=https://git-scm.com&sz=32)

Patching+1

### Practice these commands in a test repository

I recommend creating a separate GitHub practice repository and deliberately simulating these situations.

### Your advanced Git practice tracker

0/11 completed

Rebase a feature branch onto develop.

Squash five commits into one using interactive rebase.

Undo two local commits using reset --soft.

Compare reset --soft, reset --mixed and reset --hard.

Cherry-pick a hotfix commit into a release branch.

Resolve a cherry-pick conflict.

Revert an already-pushed commit.

Recover a commit using reflog.

Amend the latest commit to include a forgotten file.

Use bisect to identify a deliberately introduced bug.

Stash unfinished work, switch branches and restore it.

Reset progress
