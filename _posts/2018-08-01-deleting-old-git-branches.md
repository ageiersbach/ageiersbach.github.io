---
layout: post
title: 'Deleting old git branches'
tags: git bash
date: 2018-08-01
---

After months of work, `git branch` can return many, many branches and it
can be difficult to decide where to begin pruning. A script to find and
delete branches that hadn't been touched in say, 6 months, could backfire
because it could accidentally delete a branch on both my computer and the
remote that was shared--like *master*.

So the key thing I wanted to keep was the manual choice of deleting the branch;
after that I wanted it to be deleted from both my computer and the remote branch.
The first step is finding out which branches are stale.


If your version of git is >= 2.7.0,
it is possible to sort the branches by commit date: `git branch --sort=-committerdate`
(negative will return in descending order, positive in ascending).
If that's not available, it's possible to accomplish the same
thing with `git for-each-ref --sort=-committerdate refs/heads`, even though the
output will be different. The output can be formatted, however, to better
match `git branch` by adding the `--format="%(refname:short)"` flag. Other
formatting options that could be helpful include `"%(committerdate:relative)"`
and `"%(contents:subject)"`. Finally, it is possible to add the entire command
to your gitconfig as an alias. I appended `column -t -s '|'` to the command
so it would output the different fields as columns.

Now that the list of branches with their commit date is visible, I could begin
deleting the oldest branches--both locally and on the origin. The commands for
that are:

```
git branch -D <branchname>
git push origin --delete <branchname>
```

This could get tedious pretty quickly, so I wrote a quick function in my .bashrc file
to delete a bunch of branches at once:

```
function gb_del {
  if [[ "$#" -eq 0 ]]
  then
    echo "No branch given"
  else
    for branch in "$@"
    do
      echo "Deleting remote branch $branch"
      git push origin --delete $branch
      git branch -D $branch
    done
  fi
}
```

If I call this command as `gb_del branch1 branch2` and the first branch does
not have a corresponding branch on my remote origin, then the command would
fail and branch2 would not get
deleted. To solve this, I check if there was a remote branch with `git ls-remote`,
piping the output to `wc -l` to get the number of lines output from that command--
it'll return 0 if there is no remote branch, and 1 if there is. It's then piped
to `bc`, or [_basic calculator_](https://en.wikipedia.org/wiki/Bc_(programming_language)),
which then transforms the output of [`wc`](https://en.wikipedia.org/wiki/Wc_(Unix)), which
included spaces, to a number. Then I could check if the output of `remote_branch`
equaled 1, and delete the remote branch too.


```
function gb_del {
  if [[ "$#" -eq 0 ]]
  then
    echo "No branch given"
  else
    for branch in "$@"
    do
      local isremote=$(remote_branch $branch)
      if [[ $isremote -eq 1 ]]
      then
        echo "Deleting remote branch $branch"
        git push origin --delete $branch
      fi
      git branch -D $branch
    done
  fi
}

function remote_branch {
  git ls-remote --heads origin $1 | wc -l | bc
}
```
