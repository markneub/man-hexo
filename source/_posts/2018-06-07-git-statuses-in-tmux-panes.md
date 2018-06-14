---
title: Git Statuses in tmux Panes
date: 2018-06-07 17:00:19
tags:
 - workspace
---

As a developer, I'm often working on the command line. Last year, a [friend](http://blog.yangmillstheory.com/) at work introduced me to a terminal utility called [tmux](https://en.wikipedia.org/wiki/Tmux), and it's quickly become one of my favorite tools.

If you're not familiar with tmux, it's a program that makes managing multiple terminal sessions much easier. Instead of having a bunch of terminal windows that get lost in the mess of everything else you have open, you just have one terminal window open that can quickly access multiple prompts. You work with different prompts by using keyboard commands to configure and jump between tmux sessions, windows, and panes. A tmux session needs at least one window, but the number of panes you divide each window into is up to you. If you frequently run the same set of commands to start your development environment, you can script tmux to create a new session that automatically divides your terminal the way you like it and runs the appropriate commands. I'm barely scratching the surface here — if you're not familiar with tmux and you spend a decent amount of time at the command line, I recommend giving it a shot. I got started with (and am still using) [Oh My Tmux!](https://github.com/gpakosz/.tmux), but there are a ton of other resources on the web.

<!-- more -->

Many devs find it useful to display git status in the terminal, if the current working directory is a git repository. It's common to show this info and the current working directory using a custom shell prompt, like what you might get with a [ZSH theme](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes). There are a couple things about this solution that I don't love. First, the prompt is only accurate at the time that it's drawn to the screen. For git status, this is a problem as my current git workflow involves both the command line and a GUI; I do simple stuff in the terminal and confusing stuff like interactive rebasing in [SourceTree](https://www.sourcetreeapp.com/). If I work on the git repo outside of the terminal or in a different tmux pane, the last printed status is no longer correct. Sure, I can just hit enter to make a new prompt, but I'd be happier if that information could just update automatically. The second problem with the custom prompt solution is that the the terminal window [fills up with old prompts](https://github.com/robbyrussell/oh-my-zsh/wiki/External-themes#agnosterzak), and things quickly look cluttered. Having a rich history of commands immediately visible is valuable to some, but I don't personally find it that useful and would prefer visual simplicity.

tmux has a configurable status bar that can be placed at the bottom or top of the terminal and is nice for showing off the list of tmux windows you have open in the current session, as well as general system information like uptime, username, hostname, etc. Unfortunately, it can't be used with individual panes, which stops it from being useful for git status or current working directory, since those can differ by pane. One possible solution is to use nested tmux sessions (i.e., start another instance of tmux inside a tmux pane) and then configure the inner status bar appropriately, but this sounds complicated to configure. Since tmux 2.3, there has been a new configuration feature called `pane-border-format`, which lets the user configure some per-pane text to be placed in the bottom or top border of each pane. The [built-in variables](http://man7.org/linux/man-pages/man1/tmux.1.html#FORMATS) that `pane-border-format` accepts include `pane_current_path`, but there's no way to retrieve git status. However, it does allow printing the output of an external command. I wrote a short shell script that generates a status line with the current working directory and a git status indicator (borrowing heavily from the [bureau ZSH theme](https://github.com/robbyrussell/oh-my-zsh/blob/master/themes/bureau.zsh-theme) for the git functions) and passed it to `pane_current_path` in my tmux config. Now I have the current working path and git information updating automatically in one place for each pane, and I get to have a minimal prompt.

One downside of this approach may be increased power usage and CPU overhead of running these git commands every second, for each pane. But even with a bunch of panes open, it didn't noticeably affect power or CPU consumption on my laptop.

A brief screencast and my code are below. You could of course modify the script to show other information you might want on a per-pane basis: active nodeJS version, ssh details, etc.

<script src="https://asciinema.org/a/f0zKldxWMkTQa9mgjJ767JwVf.js" id="asciicast-f0zKldxWMkTQa9mgjJ767JwVf" async></script>
{% asset_img preview-fallback.png "asciinema fallback" %}

{% codeblock .tmux.conf.local %}
set -g pane-border-status bottom
set -g pane-border-format '#(sh ~/.dotfiles/pane-border-format.sh \
  --pane-current-path=#{pane_current_path} \
  --pane-active=#{pane_active})'
set -s status-interval 1
{% endcodeblock %}


{% codeblock pane-border-format.sh lang:bash %}
#!/bin/bash

# color variables
INACTIVE_BORDER_COLOR='#444444'
ACTIVE_BORDER_COLOR='#00afff'
RED='#d70000'
YELLOW='#ffff00'
GREEN='#5fff00'

# read args
for i in "$@"
do
case $i in
    --pane-current-path=*)
    PANE_CURRENT_PATH="${i#*=}"
    shift # past argument=value
    ;;
    --pane-active=*)
    PANE_ACTIVE="${i#*=}"
    shift # past argument=value
    ;;
    *) # unknown option
    ;;
esac
done

# replace full path to home directory with ~
PRETTY_PATH=$(sed "s:^$HOME:~:" <<< $PANE_CURRENT_PATH)

# calculate reset color
RESET_BORDER_COLOR=$([ $PANE_ACTIVE -eq 1 ] && echo $ACTIVE_BORDER_COLOR || echo $INACTIVE_BORDER_COLOR)

color () {
  INTENT=$1
  echo $([ $PANE_ACTIVE -eq 1 ] && echo $INTENT || echo $INACTIVE_BORDER_COLOR)
}

# git functions adapted from the bureau zsh theme
# https://github.com/robbyrussell/oh-my-zsh/blob/master/themes/bureau.zsh-theme

ZSH_THEME_GIT_PROMPT_PREFIX="["
ZSH_THEME_GIT_PROMPT_SUFFIX="] "
ZSH_THEME_GIT_PROMPT_CLEAN="#[fg=$(color $GREEN)]✓#[fg=$RESET_BORDER_COLOR]"
ZSH_THEME_GIT_PROMPT_AHEAD="↑"
ZSH_THEME_GIT_PROMPT_BEHIND="↓"
ZSH_THEME_GIT_PROMPT_STAGED="#[fg=$(color $GREEN)]●#[fg=$RESET_BORDER_COLOR]"
ZSH_THEME_GIT_PROMPT_UNSTAGED="#[fg=$(color $YELLOW)]●#[fg=$RESET_BORDER_COLOR]"
ZSH_THEME_GIT_PROMPT_UNTRACKED="#[fg=$(color $RED)]●#[fg=$RESET_BORDER_COLOR]"

git_branch () {
  ref=$(command git symbolic-ref HEAD 2> /dev/null) || \
  ref=$(command git rev-parse --short HEAD 2> /dev/null) || return
  echo "${ref#refs/heads/}"
}

git_status () {
  _STATUS=""

  # check status of files
  _INDEX=$(command git status --porcelain 2> /dev/null)
  if [[ -n "$_INDEX" ]]; then
    if $(echo "$_INDEX" | command grep -q '^[AMRD]. '); then
      _STATUS="$_STATUS$ZSH_THEME_GIT_PROMPT_STAGED"
    fi
    if $(echo "$_INDEX" | command grep -q '^.[MTD] '); then
      _STATUS="$_STATUS$ZSH_THEME_GIT_PROMPT_UNSTAGED"
    fi
    if $(echo "$_INDEX" | command grep -q -E '^\?\? '); then
      _STATUS="$_STATUS$ZSH_THEME_GIT_PROMPT_UNTRACKED"
    fi
    if $(echo "$_INDEX" | command grep -q '^UU '); then
      _STATUS="$_STATUS$ZSH_THEME_GIT_PROMPT_UNMERGED"
    fi
  else
    _STATUS="$_STATUS$ZSH_THEME_GIT_PROMPT_CLEAN"
  fi

  # check status of local repository
  _INDEX=$(command git status --porcelain -b 2> /dev/null)
  if $(echo "$_INDEX" | command grep -q '^## .*ahead'); then
    _STATUS="$_STATUS$ZSH_THEME_GIT_PROMPT_AHEAD"
  fi
  if $(echo "$_INDEX" | command grep -q '^## .*behind'); then
    _STATUS="$_STATUS$ZSH_THEME_GIT_PROMPT_BEHIND"
  fi

  echo $_STATUS
}

git_prompt () {
  local _branch=$(git_branch)
  local _status=$(git_status)
  local _result=""
  if [[ "${_branch}x" != "x" ]]; then
    _result="$ZSH_THEME_GIT_PROMPT_PREFIX$_branch"
    if [[ "${_status}x" != "x" ]]; then
      _result="$_result $_status"
    fi
    _result="$_result$ZSH_THEME_GIT_PROMPT_SUFFIX"
  fi
  echo $_result
}

# final output
echo " $PRETTY_PATH $(cd $PANE_CURRENT_PATH && git_prompt)"
{% endcodeblock %}