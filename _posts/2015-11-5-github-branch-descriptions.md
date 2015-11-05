---
layout: post
title:  "Git Branch Descriptions"
tags: git bash
date: 2015-11-5
---

I came across this feature in git the other day and was immediately struck by how useful it was.

First of all, from the documentation on `git branch`:

> **--edit-description**
>
> Open an editor and edit the text to explain what the branch is for, to be used by various other commands (e.g. format-patch, [request-pull](https://git-scm.com/docs/git-request-pull), and [merge](https://git-scm.com/docs/git-merge) (if enabled)). Multi-line explanations may be used.

I haven't seen this use case in action (it appears not to replace the description of pull requests on Github for multi-commit pull requests), but I had another use in mind.

Sometimes I'll be working in a branch named after a ticket, like: `ag/dd-21`, but it depends on another branch, which might not be one of mine. I usually keep these notes written down somewhere, but if I'm working away from my desk, I sometimes don't have access to all of my notes! A way I've solved this is to have a `.notes` file that is globally ignored, so that I can track all of the information about my in-flight branches. Still, I have to go hunting around for this information, which means that I sometimes forgot to check and end up doing stupid things in that branch, like rebasing on the wrong branch.

To add a branch description, you would use this command:

{% highlight text %}
=> git branch --edit-description
{% endhighlight %}

This launches the terminal's default editor, where you can enter a description like: "base on js/dd-20!" The command to show a branch description is `git config branch.branch-name.description`, or in this case:

{% highlight text %}
=> git config branch.ag/dd-21.description
base on js/dd-20!
{% endhighlight %}

This was pretty cool, but even with an alias, it would get old trying to find out what that description was after a while. So I opened up my `~/.bash_profile` and started hacking away. I came across **[this nice gist](https://gist.github.com/stuarteberg/4446426)** while I was putting this together, which gave me an idea for improving my prompt. I wanted it to look something like:

{% highlight text %}
ruby-2.0.0 wookiebookie ag/dd-21
base on js/dd-20!
[12:08 PM] $
{% endhighlight %}

So I added the following functions to my profile: `parse_git_branch()`, `parse_git_branch_color()`, and `parse_git_description()`:
{% highlight text %}
parse_git_branch() {
  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* //'
}

parse_git_branch_color() {
  br=$(parse_git_branch)
  gitstatus=`git status 2> /dev/null`

  if [[ `echo $gitstatus | grep "Changes to be committed"` != "" ]]
  then
    echo -e "${hot_pink}"
  elif [[ `echo $gitstatus | grep "Changes not staged for commit"` != "" ]]
  then
    echo -e "${yellow}"
  elif [[ `echo $gitstatus | grep "Untracked"` != "" ]]
  then
    echo -e "${red}"
  elif [[ `echo $gitstatus | grep "nothing to commit"` != "" ]]
  then
    echo -e "${green}"
  else
    echo -e "${blue}"
  fi
}

parse_git_description() {
  git config branch.$(parse_git_branch).description
}
{% endhighlight %}

I then used them in my PS1:

{% highlight text %}
export PS1="$ps1_red$(~/.rvm/bin/rvm-prompt i v g) $ps1_blue\W \[\$(parse_git_branch_color)\]\$(parse_git_branch)\n$ps1_red\[\$(parse_git_description)\]\\n$ps1_blue[\@] \$$ps1_txt "
{% endhighlight %}

... And voila! My shell now shows the branch description on the second line. If I'm not in a directory with git installed, it just shows a blank line, but that isn't bothering me at the moment.
