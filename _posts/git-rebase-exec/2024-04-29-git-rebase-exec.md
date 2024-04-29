---
layout: post
title: Git Rebase Exec
date: 2024-04-29 20:47:00 +0100
description: Git rebase's secret weapon - the exec command
tag:
  - tips
  - git
---

Git has a plethora of features, some of which are commonly used by developers like 'commit', 'push', 'pull', 'merge', etc. However, there are certain features and commands that are comparatively less known and utilized by the programming fraternity. One such command is the `exec` in `git rebase`.

Using `git rebase`, you can modify earlier or multiple commit messages, combine several commits into one, or delete or revert commits you no longer need. The `exec` command in `git rebase`, however, allows you to append a shell command after each line creating a commit in the final history.

Essentially, it attaches a shell command to be run every time a commit is applied during the rebase operation. If any attached command fails, the rebase operation is immediately stopped, allowing you to inspect the issue and figure out what went wrong.

To illustrate, you could execute the following while rebasing:

```
git rebase -i --exec "npm test"
```

This would run `npm test` after each commit is applied. If the test fails at a certain commit, you'd know immediately that something in that particular commit broke the build or functionality, making it easier to identify where exactly the error originated. Handy, no?

If you would like to run multiple shell commands, you can make use of either of the two ways:

```
git rebase -i --exec "cmd1 && cmd2 && ..."
```

This executes multiple commands for one instance of `--exec`.

```
git rebase -i --exec "cmd1" --exec "cmd2" --exec ...
```

This applies each command to more than one `--exec`.
