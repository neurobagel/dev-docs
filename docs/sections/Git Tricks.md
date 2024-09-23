---
title: Git Tricks

---

# Things we've learned
This is a place to keep solutions we find / learned when dealing with git problems (or the references that have the answers).

## Github specific

### My PR has too many changes / merging into the wrong branch
PRs by default are made against the default branch (`main` / `master`). You can select a different one when you create the PR, or change it later. See here: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/changing-the-base-branch-of-a-pull-request

### I want to partially revert a commit
You made some changes, but got too excited and lumped together a bunch of things in one commit that aren't really linked. Classic case: reformatted the file and added some functional changes. Your observant reviewer says to please revert the formatting (but keep functional changes). Now what?

Step 1: `git revert <offending commit hash>`. See here https://www.atlassian.com/git/tutorials/undoing-changes/git-revert. Now you have created a new (revert) commit that undoes all what the offending commit has introduced. That's good, but now the part of the commit we liked is also undone. (note: do not use rebase here, we don't want to rewrite history)

Step 2: `git checkout -p <offending commit hash>`. See here https://git-scm.com/docs/git-checkout#Documentation/git-checkout.txt---patch This checks out the offending commit (from further down in the git log). The `-p` flag makes this an interactive process: your default text editor will show you the changes in the offending commit, one (c)hunk at a time and you can then decide whether you'd like to apply that change (again) or not. The way you use this in our example is to say yes to the functional changes, and no to the formatting changes.

At the end of this process, you have new staged changes (the ones you said yes to during the patch checkout) and you can now either --amend your revert commit (to make it only a partial revert) or make a new commit for these specific changes.

### Cloning a private repo without ssh key
You may need to do this in order to clone a private data repo to one of our shared remote cloud servers, since any ssh key files copied to these servers could be visible to team members using the same account.

First, on the remote server where you need to clone the repo, login to GitHub using the gh CLI:
`gh auth login`

Choose HTTPS as the preferred protocol for Git operations, and then select `Login with a web browser` when prompted to authenticate GitHub CLI.

Copy the one-time code provided, and press Enter. Then, in a browser on your **local machine**, copy the URL provided (https://github.com/login/device) and enter the one-time code.


# Good resources
- https://dangitgit.com/ (list with solutions to common problems)
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials) (very well made tutorials, from beginner to advanced)
- [Offical git docs](https://git-scm.com/docs) (bit technical)