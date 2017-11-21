---
title: zshrc
date: 2017-11-14 13:35:28
categories: IT
tags: [mac]
---
# zshrc  文件备份
```
# Correctly display UTF-8 with combining characters.
if [ "$TERM_PROGRAM" = "Apple_Terminal" ]; then
	setopt combiningchars
fi
[[ -s ~/.oh-my-zsh/plugins/autojump/autojump.plugin.zsh ]] && . ~/.oh-my-zsh/plugins/autojump/autojump.plugin.zsh
[[ -s ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh ]] && . ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
disable log
# Generic
alias ls='ls -G'
alias su='su -'
alias ll='ls -l'
alias la='ls -a'
alias l='ls -lF'
alias df='df -h'

# Git
alias ga='git add'
alias gb='git branch'
alias gba='git branch -a'
alias gbd='git branch -d'
alias gcam='git commit -a -m'
alias gcb='git checkout -b'
alias gco='git checkout'
alias gcp='git cherry-pick'
alias gd='git diff'
alias gfo='git fetch origin'
alias ggpush='git push origin $(git_current_branch)'
alias ggsup='git branch --set-upstream-to=origin/$(git_current_branch)'
alias glgp='git log --stat -p'
alias gm='git merge'
alias gpush='git push'
alias gst='git status'
alias gsta='git stash save'
alias gstp='git stash pop'
alias gpull='git pull'

# mysql
alias mysql='/usr/local/mysql/bin/mysql'
alias mysqladmin='/usr/local/mysql/bin/mysqladmin'

# maven
alias mi='mvn clean install -Dmaven.test.skip=true'
alias mp='mvn clean package -Dmaven.test.skip=true'
alias mdp='mvn clean package deploy -Dmaven.test.skip=true'

alias vim='sudo vim'


[ -r "/etc/zshrc_$TERM_PROGRAM" ] && . "/etc/zshrc_$TERM_PROGRAM"
```
