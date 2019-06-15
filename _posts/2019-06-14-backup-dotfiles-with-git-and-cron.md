---
layout: post
title:  "Automatically back up dotfiles and config with git and cron"
date:   2019-06-14 19:20:04 -0700
description: Discussion of an automated backup strategy for configuration files to make restoring from backups or purchasing a new computer easy.
author: Thomas Danner
lang: en_US
categories: devtips
tags: backups, dotfiles, git, cron, crontab, shell
---

# Automated dotfiles backup with git and cron

If you have been coding for any amount of time, you have probably experienced hard drive failure, computer theft, or liquid damage to your precious machine. Maybe, infallible developer though you are, one day you just didn't look before you rm -rf'd, and now you are well and truly rm -r f'd.

What to do?

## Rules of backups for the lazy programmer
1. If it requires you to physically do something, it's not going to happen. You are a busy professional with just as many best intentions as you have temporal limitations. You will remember it today, maybe for the next week, but you will inevitably forget. So requirement #1 is automated tracking of files you wish to keep. Plus, computers are designed to do this work for us.
2. Restoration should be as simple as downloading a script, pressing enter, and maybe entering your password a couple of times.

## How I do it

In short:

0. dotfiles and their updates over time are stored in a git repository.

1.  my machine's crontab runs a script each hour that checks in any changes from a list of files that I specify (.vimrc, .bashrc, etc) into the configuration repo. It also updates things like the current list of all my brew packages and vscode extensions that are installed. If there are any changes, it automatically git commits and pushes them to my remote repo  repo that holds the current state of my config

2. a provisioning script for a new computer pulls the config repo, restores the dotfiles, and reinstalls all the software packages and editor fixings I had on my machine, down to the last time my repo was updated

On a Mac, I use [brew](https://brew.sh/) fairly extensively to handle package management. Brew has a nice feature called `brew bundle`, which enables you to make a list of all the current packages installed on the machine. `brew bundle dump` creates the list, and `brew bundle install` reinstalls every package on that list.

Storing updates in git is also nice, because if you happen to mess up your configuration somehow, chances are you can track down the buggy commit in git and revert the changes.

Also note that just because you are uploading your config to a "private repo", it would be an *exceedingly awful idea* to use this method for things like ssh key management, or really anything at all that you wish to remain remotely private.

## Here is what my script looks like.

### provisioning script excerpt
```bash
#!/bin/bash

# this script is downloaded from the config repo and run on
# a new machine to reinstall all packages, configure dotfiles,
# and set up preferences

# secret, set up my SSH

# secret, set up git

# .dotfiles paths
BACKUP_DIR="$HOME/.BACKUP_CONFIG"
DOTFILES_DIR="$BACKUP_DIR/dotfiles"
BREW_INSTALLS_FILE="$BACKUP_DIR/brew_bundle_dump"
VSCODE_EXTENSIONS_FILE="$BACKUP_DIR/vscode_extensions"

# Clone my dotfiles repo: $BACKUP_DIR is my local folder where
# backup copies of all my dotfiles and config are saved.
git clone --depth=1 git@github.com:ME/MYSECRETREPO.git "$BACKUP_DIR" || {
  printf "Error: git clone of configuration repo failed\n"
  exit 1
}
# Copy dotfiles into homedir
cp -a "$DOTFILES_DIR" "$HOME"
git config --global core.excludesfile "$DOTFILES_DIR/.gitignore_global"

# Install Brew
if test ! "$(command -v brew)"; then
  echo "Installing homebrew..."
  ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
fi

export HOMEBREW_NO_ANALYTICS=1
export HOMEBREW_NO_INSECURE_REDIRECT=1
export HOMEBREW_CASK_OPTS=--require-sha

# Install all brew packages
brew update-reset && brew update
brew tap Homebrew/bundle
brew bundle install --file="$BREW_INSTALLS_FILE"
brew upgrade --all && brew cleanup

# Set up packages just installed via homebrew

## vscode
### install extensions
while IFS= read -r line; do
  code --install-extension "$line"
done < "$VSCODE_EXTENSIONS_FILE"

### import settings
VSCODE_SETTINGS_FOLDER="$HOME/Library/Application Support/Code/User/"
mkdir -p "$VSCODE_SETTINGS_FOLDER"
mv "$DOTFILES_DIR/vscode_settings.json" "$VSCODE_SETTINGS_FOLDER/settings.json"

# bootstrap my backup script
"$BACKUP_DIR/config_updater.sh"
```

And, the backup script, referenced at the end of the provisioning script...

### `config_updater.sh`

```bash
#!/bin/bash
# This script commits any local configuration changes and automatically
# pushes them to the remote.
#
# Uploads newest versions of dotfiles from the $HOME dir into
# $HOME/.BACKUP_CONFIG, a folder created by the provisioning script
# that is linked to the remote configuration repo.

BACKUP_DIR="$HOME/.BACKUP_CONFIG"
THIS_SCRIPT_FULL_PATH="$BACKUP_DIR/$(basename -- "$0")"
DOTFILE_DIR="$BACKUP_DIR/dotfiles"
mkdir -p "$BACKUP_DIR"
mkdir -p "$DOTFILE_DIR"

BREW_INSTALLS_FILENAME="brew_bundle_dump"
VSCODE_EXTENSIONS_FILENAME="vscode_extensions"

/usr/local/bin/brew bundle dump --force --file="$BACKUP_DIR/$BREW_INSTALLS_FILENAME"
/usr/local/bin/code --list-extensions > "$BACKUP_DIR/$VSCODE_EXTENSIONS_FILENAME"
dotfile_list=("$HOME/.gitignore_global" "$HOME/.bashrc" "$HOME/.vimrc")

for file in "${dotfile_list[@]}"; do cp "$file" "$DOTFILE_DIR"; done

cd "$DOTFILE_DIR" || exit
if ! git diff --quiet HEAD || git status --short; then
  git add --all
  git commit -m "updating dotfiles on $(date -u)"
  git push origin master
fi

# Make this script call itself hourly from the crontab, if it isn't already.
if ! crontab -l | grep "$THIS_SCRIPT_FULL_PATH"; then
  (crontab -l ; echo "0 * * * * $THIS_SCRIPT_FULL_PATH > /dev/null 2>&1") | sort - | uniq - | crontab - 2>&1
fi
```

If you're not familiar with crontab, [read up on it here](https://en.wikipedia.org/wiki/Cron). Aside from slightly unfamiliar syntax for defining times, it's basically a big to-do list for your computer for recurring tasks. It's very useful, easy to learn, and almost every server everywhere is probably running one. Before you know it, you'll be writing down `12 * * * * ./eat_lunch.sh` on your calendar for lunch dates.

Hopefully this article has given you some ideas to inspire your own backup strategy. It's worth the initial time investment. Just think about how many little things you've customized on your machine, and imagine having to restore them all (much less: remember them all!) by hand, following a hardware failure.