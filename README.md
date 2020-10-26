# Git

This guide assumes Git version 2.28.0.
You can install it from [Homebrew](https://brew.sh/) with `brew install git`.
The version of Git that comes with macOS is older, so if some of these commands don't work, check `git --version`.

## Core Concepts

### Commits

A commit is the core unit of change in Git.
Each commit adds or removes lines in files.

<img src="structure-of-a-commit.png" width="639" />

Every commit has a hexadecimal hash (`36ee8abf2`...) that uniquely identifies it among all the other commits in the repository.
`git log` shows the full 40-character hash for each commit.
Since 40-character hashes are cumbersome, GitHub often shows just the first 6-9 characters of the hash (`36ee8ab`), just enough to identify it uniquely, and hides the rest.

Commits point to their parent commit.
For example, if I make three commits in a repository, A &larr; B &larr; C, C would be the most recent commit, B its parent, and C its grandparent.
`git log` shows the most recent commit and all of its parents.

Git can reconstruct the repository at any point in time by applying the sequence of added and removed lines in the commit chain.
When we make changes by adding commits or merging pull requests, Git looks at those commits to know which lines to add and remove.

### `HEAD`

In Git, `HEAD` is a special alias for the most recent commit.
It's whichever commit `git log` would show first.
When we create a new commit, the previous `HEAD` commit is the new commit's parent, and then `HEAD` is updated to point to the new commit.

### Branches

Branches are special labels that point to commits.
`master` is a regular branch whose label is "master".
Sometimes repositories choose to call it `main` instead.
Because we start work by branching from `master`, our branches look like a tree with `master` as the trunk.
We can follow commits' parents in a chain and eventually end up back in the `master` trunk.

<img src="branches.png" width="620" />

We can create our own branches with `git branch new-branch-name`.
This will create a new label pointing to the most recent commit.

We can switch between branches with `git switch other-branch`.

When we create a new commit, the current branch's label automatically updates from the previous commit to the new commit.

### Pushing and pulling

Our laptops, GitHub, and each Heroku environment all have their own copy of our repository.
These repositories keep track of branches and commits separately.
We use `git push` and `git pull` to copy commits and branch labels between repositories.

`git remote --verbose` lists all of the other repositories that Git knows about.
GitHub is typically called `origin`, and `production` and `staging` might be names for Heroku's repositories.

When we push a new branch to GitHub, it pushes all of the commits we've made locally and then tells GitHub to create its own copy of the branch label pointing to the most recent commit.

When we push new commits to an existing branch in GitHub, Git sends GitHub all the new commits that are in our laptop's branch but not yet in GitHub's branch and updates GitHub's branch label to point to the newest commit.

Pulling works in reverse.
When we pull a branch from GitHub, Git retrieves all the commits that are in GitHub but not yet on our laptop and updates our laptop's branch label to point to the newest commit.

## Working on code

The `master` branch should be the authoritative source for what exists in production.
Changes should only reach `master` when the tests are passing and they're ready for production.

### Starting a feature or bug fix

We work on in-progress changes in "feature branches" off of `master`.
We feature branch might implement a new feature, add a test, change some copy, or fix a bug.

```sh
# Make sure we're starting our branch off of master.
$ git switch master
# And make sure we have the latest changes from master on GitHub.
$ git pull
# Create the feature branch from master. Name it something descriptive, like
# `very-cool-feature` or `fix-issue-123`.
$ git branch new-branch-name
# Switch to the new branch.
$ git switch new-branch-name
```

At this point, we're working on the `new-branch-name` branch.
For now, `HEAD`, `new-branch-name`, and `master` all point to the same most recent commit.
Any new commits will be added to `new-branch-name` but not `master`.

### Committing changes

Once we've made a change, we use `git add` to prepare the added and removed lines to be turned into a commit.
When we `git add` a file, it goes into a "staging" area, which is like a waiting room for the lines that will be in a commit.

```sh
# Show what files have been changed since the last commit.
$ git status
# Add a file.
$ git add hello-world.js
# Show that hello-world.js has been added to the commit staging area.
$ git status
# Turn the staged files into a commit.
$ git commit
```

The `git commit` command will open a text editor where you can write a title and message for your commit.
The title is a short description in a few words.
The message contains a more detailed description of what changes we made and why.
If we're adding a new feature, we describe the feature's design and how it's built.
If we're fixing a bug, we describe what caused the bug and why this is the right way to fix it.

### Opening a pull request

Congrats!
We've implemented a new feature or fixed a bug.
All the steps in the change are nicely packaged as line changes in commits.
It's time to open a pull request in GitHub.

```sh
# Tell GitHub about our new commits and branch.
$ git push --set-upstream origin my-new-branch
```

`git push` copies all of the new commits to GitHub and creates a copy of our branch label.
`origin` is Git's name for GitHub's copy of our repository.
`--set-upstream` tells Git to remember that we pushed `my-new-branch` to GitHub, so in the future, just `git push` will automatically go to GitHub.
Now go to GitHub and fill out your pull request.

#### Review feedback

We've received review feedback on the pull request and need to make changes.
Make commits just like when we were working on the change initially.

```sh
# Show the files we've updated in response to feedback.
$ git status
# Add them to the staging area for the next commit.
$ git add hello-world.js
# Commit our review fixes.
$ git commit
# Push the fixed commits to GitHub.
$ git push
```

Git will send our review feedback changes to GitHub, update GitHub's `my-new-branch` branch label to point to the new commits, and show them in the pull request.
In the `push` step, since we used `--set-upstream` when we first pushed the branch to GitHub, we don't need to specify `origin my-new-branch` again because Git remembers those settings.
It's the same as running `git push origin my-new-branch`, just with less typing.

#### Suggested changes

Sometimes reviewers leave small suggested changes in review comments.
Committing these suggestions to the pull request is quick and easy without ever leaving GitHub.
After you commit review suggestions from GitHub, be sure to update your local repository so it knows about the new commits.

```sh
# Switch to the pull request's branch.
$ git switch my-new-branch
# Update the local branch with the suggested change commits from GitHub.
$ git pull
```

#### Rejected push

If we forget to update our local branch when there are new commits in GitHub's copy of the branch, Git will reject our next push to the pull request.
**Never** run `git push` with `--force`.
That would overwrite the suggested change commits from GitHub, losing work.
Git rejected the push because there are commits in GitHub's copy of `my-new-branch` that are missing from our local copy of `my-new-branch`.
We want to combine the commits from all versions of the branch.
Run `git pull --rebase` to take the suggested change commits from GitHub and then replay our local commits after them.

<img src="rebase-suggested-change.png" width="798" />

```sh
# Git rejects our push because our local branch is missing some commits.
$ git push
To https://github.com/drivecapital/git-workflow.git
 ! [rejected]            my-new-branch -> my-new-branch (non-fast-forward)
error: failed to push some refs to 'https://github.com/drivecapital/git-workflow.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
# Pull the newer commits from GitHub and replay our local commits after them.
$ git pull --rebase
# Now we can push the combined commits to GitHub.
$ git push
```

### Merge conflicts

Sometimes GitHub says we can't merge a pull request because we have merge conflicts.
This happens when a teammate worked on the same lines in a different branch.
Git doesn't know which version of the new lines to take - ours or our teammate's, so it asks us to decide.

First, we need to pull the latest changes from GitHub's `master` branch label to our laptop.

```sh
# Switch to our local master branch.
$ git switch master
# Retrieve the new commits from GitHub.
$ git pull
```

_TODO: Finish this section._

## Git etiquette

_TODO: A commit should change one thing, and pull requests should only include related commits. How to make things easier for reviewers._

## Advanced usage

### Reverting a pull request

If we just merged and deployed a pull request that broke everything and we can't fix it in a hurry, the revert button is the nuclear option.
Clicking the "revert" button on the offending pull request will reverse all of its changes, removing the lines it added and adding back the lines it removed.
If we do a good job submitting pull requests that make logically grouped changes and splitting unrelated changes into their own pull requests, we should be able to revert just the pull request that caused the breakage.
We shouldn't need to use the `git revert` command locally.

### Committing only part of a file

Each commit should do one thing.
If we've fixed two unrelated bugs, we'd like to create two commits, one to fix each bug.
We don't have to `git add` an entire file at once. Instead, we can add only part of a file.

```sh
# Start an interactive session to stage lines for the commit a chunk at a time.
$ git add --patch
```

`git add --patch` will go through the changes one chunk of lines at a time.
Typing `?` at the prompt will show help instructions.
`y` will stage this chunk of lines for inclusion in the next commit.
`n` will leave this chunk of lines unstaged for a later commit.
If the chunk of lines is too big, split it into even smaller chunks with `s`, then stage or skip the sub-chunks with `y` or `n`.

### Un-staging changes

If you've added a file to the commit staging area and don't want it to be part of the commit, you can remove it from the staging area before committing.

```sh
# Show the files in the staging area for the next commit. Whoops, hello-world.js
# isn't supposed to be part of this commit! Let's leave it for a later commit.
$ git status
# Remove hello-world.js from the staging area.
$ git unstage
# Show that hello-world.js has been removed from the commit staging area. A new
# commit now would not include the added and removed lines in hello-world.js.
$ git status
```

### Interactive rebase

_TODO_

## Configuration

When pulling from GitHub, only allow "fast forwards".
When "fast forwarding", if your local branch contains commits A, B, C, and GitHub has A, B, C, D, E, pulling will add D and E to your local branch.

```sh
$ git config --global pull.ff only
```

Enable branch protection in your GitHub repository's settings:

- `master`
    - Don't allow force pushes
        - This only applies to `master`.
        - We can still `--force-with-lease` to feature branches in advanced uses.
    - Don't allow deletions
    - Include administrators
        - Prevents even organization owners from accidentally force pushing to `master`.
        - If 💩 hits the fan and you really need to, you can temporarily un-check this.
    - Optionally, require pull request reviews before merging
        - Changes must go through code review.
        - Prevents accidentally pushing directly to `master`.
    - Optionally, require status checks to pass before merging
        - If your automated tests aren't flaky.
    - Optionally, require linear history
        - Forces people to resolve conflicts via rebasing their feature branch on `master` instead of merge commits.