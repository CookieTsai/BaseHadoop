# Install Homebrew — The missing package manager for OS X

	注意事項：
		1. 需要使用到 ruby 請確保安裝 homebrew 前已裝好
		2. 內容主要目的為方便查閱，內容整理自官網及手冊文件中
		3. 文章僅供參考

## Descreption

	Homebrew is the easiest and most flexible way to install the UNIX tools Apple didn't include with OS X.

## Install

	$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
	
## version

	$ brew -v

## help

	Example usage:
	  brew [info | home | options ] [FORMULA...]
	  brew install FORMULA...
	  brew uninstall FORMULA...
	  brew search [foo]
	  brew list [FORMULA...]
	  brew update
	  brew upgrade [--all | FORMULA...]
	  brew pin/unpin [FORMULA...]

	Troubleshooting:
	  brew doctor
	  brew install -vd FORMULA
	  brew [--env | config]

	Brewing:
	  brew create [URL [--no-fetch]]
	  brew edit [FORMULA...]
	  open https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/Formula-Cookbook.md

	Further help:
	  man brew
	  brew home

## Links

* [Homebrew 官網連結](http://brew.sh/)
* [Homebrew github](https://github.com/Homebrew/homebrew)