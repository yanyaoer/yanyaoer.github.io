---
title: my osx setup
date: 2013-06-09 12:05:50
---

#setting
```
# Enable full keyboard access for all controls
defaults write NSGlobalDomain AppleKeyboardUIMode -int 3

# Disable menu bar transparency
defaults write NSGlobalDomain AppleEnableMenuBarTransparency -bool false

# Allow quitting Finder via ⌘ + Q; doing so will also hide desktop icons
defaults write com.apple.finder QuitMenuItem -bool true

# Avoid creating .DS_Store files on network volumes
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true

# Disable the warning when changing a file extension
defaults write com.apple.finder FXEnableExtensionChangeWarning -bool false

# Enable tap to click (Trackpad)
defaults write com.apple.driver.AppleBluetoothMultitouch.trackpad Clicking -bool true

# Enable Safari’s debug menu
defaults write com.apple.Safari IncludeInternalDebugMenu -bool true

# To pin the dock to the right bottom
defaults write com.apple.dock pinning -string end
defaults write com.apple.dock orientation -string right

# Only Show Open Applications In The Dock
defaults write com.apple.dock static-only -bool true

defaults write com.apple.dock tilesize -int 24
defaults write com.apple.finder QLEnableTextSelection -bool true

# disable dashboard
defaults write com.apple.dashboard mcx-disabled -boolean true
```

#[xcode] with [Command Line Tools]
```
# xcode by appstore
open /Applications/App\ Store.app

# CUI Downloader of "Command Line Tools for Xcode"
dlclt.py [dlclt]
xcode-select --install
```

#[homebrew]

```shell
ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"

brew install coreutils fish tmux wget axel aria2 vim ctags git python3 node mysql mongodb redis
```

#pyenv
```shell
git clone git://github.com/yyuu/pyenv.git .pyenv
sudo easy_install -U setuptools #upgrade setuptools for pip
sudo easy_install pip
sudo pip install virtualenvwrapper

#optional use douban pypi mirror
vim ~/.pip/.pip.cof
[global]
index-url = http://pypi.douban.com/simple
```

#application

[brew-cask]
```shell
brew tap phinze/homebrew-cask
brew install brew-cask

brew cask install iterm2 alfred macvim sourcetree nvalt xquartz sequel-pro bettertouchtool amethyst mplayerx google-chrome controlplane dropbox bittorrent-sync colors skim geektool keyremap4macbook pckeyboardhack

#install quicklook plugins
brew cask install qlcolorcode qlstephen qlmarkdown quicklook-json qlprettypatch quicklook-csv betterzipql webp-quicklook suspicious-package

brew cask alfred
```



none-cask

  * dash
  * popclip
  * pocket


#keyboard

remap caps lock as hyperkey to toggle alfred
or hold like modify key with shift+ctrl+opt+cmd



[ref]: https://gist.github.com/erikh/2260182
[other]: https://github.com/mathiasbynens/dotfiles/blob/master/.osx
[xcode]: http://itunes.apple.com/us/app/xcode/
[Command Line Tools]: https://developer.apple.com/downloads
[homebrew]: http://mxcl.github.io/homebrew/
[brew-cask]: https://github.com/phinze/homebrew-cask
[dlclt]: https://gist.github.com/uchida/3559987
[custom osx]: http://www.wijeyesakere.com/tech/customosx/
