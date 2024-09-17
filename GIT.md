# Git cheat sheet for contributors

## TOC

1. [Preamble](#preamble)
1. [DCO](#the-dco-signoff)
1. [Signing](#signing-commits)
1. [Squashing](#how-to-squash)
1. [Avoid having to squash](#how-to-avoid-having-to-squash-amend-your-last-commit)
1. [Rebase, not merge](#rebase-do-not-merge)
1. [Manage commits sets](#rebasing-interactively-and-managing-a-set-of-individual-commits-for-large-bodies-of-work)
1. [Naming branches](#name-your-branches-in-a-consistent-manner)
1. [Naming stashes](#-name-your-stashes)
1. [FAQ](#FAQ)

## Preamble

You should be familiar with git/hub basic workflow & operations (fork, clone, commit, push, PR).
In case you need a refresher, there are plenty of good tutorials out there.

Then, contributing to open-source projects typically requires a little bit more knowledge,
and most projects routinely have an additional set of expectations about DCO, signing,
squashing, rebasing.
Furthermore, open-source projects are normally much more picky with commit history and commit messages
than your regular in-house projects.

More generally, when contributing, you will be better off being fluent in dealing with multiple branches and
rewriting history.

While not particularly advanced, this cheat-sheet aims at covering most of these questions,
hopefully helping new contributors get the git gymnastics out of the way, and freeing them to 
focus on the gist of their contributions instead of the mundanities.

## The DCO signoff

The [DCO](https://wiki.linuxfoundation.org/dco), often
required by open-source project for legal reasons, is a "per-commit sign-off" 
stating that you agree to the terms of the [Developer Certificate of Origin](https://developercertificate.org/)
for that specific commit (<- please read it - it is very short).

Adding the DCO to your commits is trivially easy - just add the `--signoff` flag:

```bash
git commit --signoff -m "My commit message"
```

Or the "short form" flag, `-s`:
```bash
git commit -s -m "My commit message"
```

## Signing commits

> note that this is not the same thing as the DCO

Cryptographically signing your commits guarantees that you are indeed the author of that code, preventing someone else from
impersonating you.

For example, if the github repository was to be compromised, they would
still be unable to "sign" commits in your name, since they would not have your key (hopefully!).

Some projects do require you to sign your commits.
Most projects consider it good practice, and there is no good reason not to do it.

Signing commits was historically done using a gpg key (gpg key management is out of scope here),
but hopefully nowadays you can just sign using your regular ssh key.

### Github configuration

- on the github website, go to "settings" >  "ssh and gpg keys" > "new ssh key"
- give it a name, then select the dropdown "key type > signing key"
- paste your PUBLIC ssh key into the textarea
- click "add ssh key"

### Local configuration

Edit your `~/.git_config` file and add the following sections:
```git
[commit]
	gpgsign = true

[gpg]
	format = ssh
```

### Signing your commits

Just add `-S` to your commit (note the capital `S` - not lower-case `s` which is for the DCO).

```bash
git commit -S -m "Message"
```

Since you are likely going to use the DCO _as well_, your commit command should really look like:

```bash
git commit -S -s -m "Message"
```

Now your commits will show on github with a nice green "verified" tag.

## How to squash

So, you did a bunch of incremental changes for one "thing", and added them as (let say "five") individual
commits instead of [amending](#how-to-avoid-having-to-squash-amend-your-last-commit).

You submitted your PR, and a maintainer is asking you to "squash".
Meaning: they want ONE commit, not FIVE.

To do that:

```bash
# Rollback the last five commits, but KEEP the changes in your local copy
git reset --soft HEAD~5
# Re-add all your changes
git add .
# Make *one* commit
git commit -S -s -m "my one commit"
# Force-push, since your remote and local branches have diverged
git push -f
```

## How to avoid having to squash: amend your last commit

Save yourself time. If you are just tweaking / iterating on one thing, do not
pile-up commits, just amend:

```bash
git add somechange
git commit --amend
# ...
# When done...
git push -f
```

## Rebase, do not merge

Fast moving projects will regularly evolve while you are still working on your PR,
or waiting for a review.

As the "main" branch drifts away, your work may become stale, or even grow conflicts.

More generally, it is just good practice to "rebase" your work regularly on top of the 
latest state of the main branch so that your work really applies to the "now" instead of
"yesterday".

To do that, do NOT "merge" branches - as this will create useless additional
commits.

Instead, just "rebase" your work on top of main.

```bash
# Let say you are currently on branch "my-shiny-branch"

# Start by refreshing your local copy of main
git checkout main
git pull
# Now, go back to your branch, and rebase it against main
git checkout my-shiny-branch
git rebase main

# If you do have conflicts, go in your code and fix them
# then add the resolved files, and continue
git add conflicted_file_resolved
git rebase --continue

# Once you are fully done with the rebase, just push again
git push -f
```

Note that rebasing will _change_ your branch history, as your modifications
will now be based on-top of different (new) commits.
This is why you have to force push, as the remote and local branches have diverged.

## Rebasing interactively and managing a set of individual commits for large bodies of work

Opposite to [please squash your commit](#how-to-squash), maintainers being faced
with large changes, or with changes addressing multiple things at once, will ask you
to split these changes into individual commits to facilitate review.

Then during review, you are being asked to change certain things that were added
in different commits in your commit pile...

Let say you have to modify `file_1` (but only a part of it) in commit number 1, remove the changes from commit number 3,
change another part of `file_1` in commit number 4, and change `file_2` in commit number 5.

Start by doing all your changes, test them, but do not commit them.

Now, stash them:
```bash
git stash
```

Let's rewrite history interactively for the past five commits:
```bash
git rebase --interactive HEAD~5
```
You will be presented with this list (oldest commit first):
```bash
pick c5d3309d Commit 1
pick 51af3690 Commit 2
pick b8b5d707 Commit 3
pick 38bca1f3 Commit 4
pick ec530147 Commit 5
```
Now, turn it into this:
```bash
# We are going to do changes in commit 1
edit c5d3309d Commit 1
# We do no touch this one
pick 51af3690 Commit 2
# We are dropping commit 3, so, comment out
# pick b8b5d707 Commit 3
# We are going to edit these two as well
edit 38bca1f3 Commit 4
edit ec530147 Commit 5
```

Save and exit.
You have now been dropped into the state of the repo at `Commit 1`.

Now, let's get our modification back from the stash:
```bash
git stash pop
```

And let's add the PART of file 1 we care about for this commit:
```bash
git add -p file_1
```

Confirm which parts you want, then amend commit 1 and move on:

```bash
git commit --amend
git rebase --continue
```

You are now in commit 4 (we left commit 2 untouched, and commit 3 was dropped).
```bash
git add -p file_1
git commit --amend
git rebase --continue
```

Then commit 5:
```bash
git add file_2
git commit --amend
git rebase --continue
```

You are done.
Push now:

```bash
git push -f
```

## Name your branches in a consistent manner

Busy contributors will routinely have to manage a (few) dozen(s) different branches
for a single repo.
Also, projects are sometimes slow to review, and you have to be patient, keeping
these branches locally for a while.
Finally, you may very well start working on something and have it only locally
for an extended period of time before you send it as a PR.

Before you know it, you will end-up with a large set of local branches, and thanks to your 
ridiculous naming non-strategy (`fix-that-shit`, `that-would-be-nice`, `tentative-cleanup`, `omg`),
you just can't tell anymore what is what, what matters, what should be deleted, or even what got merged.

Using a consistent scheme for naming will help you keep your sanity, and avoid having to file
your clone for chapter 11.

The following is not normative - just a recommendation:

```bash
git branch TYPE-ISSUE_NUMBER-SHORT_DESCRIPTION
```

By having an issue number in the branch name, you will at least be able to retrieve
the ticket, which you have (hopefully) linked to the actual PR, helping you figure out
if that got merged or not.

"Type" could be for example "bugfix", "feature", "chore" (or "cleanup", or "refactor").
Convenient for conceptually grouping branches together.

The "short description" will still help you figure out what it is at a glimpse
(eg: "implement-file-store", or "fix-broken-hook").

## ... name your stashes

```bash
# Do not:
# git stash
# git stash list
# Do
git stash save -m "there is this thing in this stash"
git stash list
```

Your future self will thank you.

## FAQ

### I already did my commit but forgot to use the DCO, how do I add it?

Just amend your commit:
```bash
git commit --amend --signoff
```

### What exactly is `--sign-off` doing?

It is very simple. It does append the following line to your commit message:

```
Signed-off-by: yourname <youremail>
```

Note that `yourname` and `youremail` are retrieved from your git configuration,
which you can see using:
```bash
git config --global user.name
git config --global user.email
```

And which you can modify with:
```bash
git config --global user.name MYNAME
git config --global user.email MYEMAIL
```

Or by editing directly your `~/.gitconfig` file:
```bash
[user]
        name = yourname
        email = youremail
```

This of course can also be configured per-project instead of globally,
for example in case you would like to use different email addresses in different contexts.

