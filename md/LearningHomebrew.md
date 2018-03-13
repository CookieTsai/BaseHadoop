# Learning Homebrew

	注意事項：
		1. 本文來源 http://blog.lyhdev.com/2011/06/homebrew-mac-os-x.html
		2. 目的為紀錄便於日後查閱翻找
		3. 文章內容僅供參考

用過 Ubuntu 或其他 Linux 的朋友，是否覺得 Mac OS X 少了些方便好用的軟體工具呢？

雖然 Mac 有很棒的 AppStore ，可以線上下載/購買安裝許多好用的軟體，不過有一些在 Linux 系統經常使用的工具，卻找不到類似 apt 或 yum 等好用的套件管理工具，可以幫忙自動安裝和移除。

例如我經常使用 wget 抓取 URL 檔案，或是使用 lftp 連到遠端伺服器上下傳資料，可惜這些在 Linux 系統很普遍使用的工具， OS X 並未內建。

幸好 OS X 和 Linux 同樣具有 Unix 的血緣，所以不少常用的工具，其實都能用原始碼進行編譯及安裝。 OS X 內建的開發環境相當不錯，提供 make 及 gcc 等需要的工具，我們只需要一套簡便的套件管理工具，協助下載原始碼及完成編譯安裝等動作。

Homebrew 是本文要推薦的工具，它使用 Ruby 語言開發，目標是成為一套簡單並且具有彈性的套件管理工具，協助使用者在 OS X 系統上安裝 Unix 程式。

Homebrew - The missing package manager for OS X

在安裝 Homebrew 之前，系統必須裝有 ruby 開發工具，一般的 Mac OS X 系統已內建 ruby ，因使用以下的指令即可安裝 Homebrew 。

	$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

安裝好 Homebrew 之後，可以用 brew 指令進行操作，常用的指令格式有四種。

**更新套件清單**
	
	$ brew update
	
**搜尋某項軟體**

	$ brew search 軟體名稱

**安裝新軟體**

	$ brew install 軟體名稱

**移除軟體**

	$ brew uninstall 軟體名稱


**例如要安裝 wget ，可以輸入：**

	$ brew install wget

就可以看到 Homebrew 自動幫忙下載原始碼壓縮檔，解壓縮，執行 ./configure 設定，編譯(make)及安裝(make install)，這真的太方便了。

如果遇到需要的軟體，已經無法下載檔案，該怎麼辦呢？

以 lftp 來說，現在就會遇到這種情況，所以會顯示以下錯誤：

	curl: (22) The requested URL returned error: 404
	
這真是太悲情了！

幸好 Homebrew 的設定真是超級簡單！超級EASY！只要修改一下設定檔就可以解決。

首先用瀏覽器打開  http://ftp.yars.free.net/pub/source/lftp/ 網址，會發現問題的原因在於，檔案名稱 lftp-4.2.2.tar.bz2 已經不存在，當然無法下載；因為 lftp 已經發佈新版本，檔名是 lftp-4.3.1.tar.bz2 。

Homebrew 針對每一種軟體套件，都有獨立的個別設定檔，稱為 FORMULA ，這些設定檔位於 /usr/local/Library/Formula 路徑下。

以 lftp 工具來說，設定檔就是 /usr/local/Library/Formula/lftp.rb ，用文字編輯器打開，可以發現設定檔其實就是 Ruby 程式。以前面所提到的問題，只需要修改兩個地方，請參考以下紅色字體標示的部份。

<pre>
require 'formula'

class Lftp < Formula

	url 'http://ftp.yars.free.net/pub/source/lftp/<font color="red">lftp-4.3.1.tar.bz2</font>'

	homepage 'http://lftp.yar.ru/'
	md5 '<font color="red">ea45acfb47b5590d4675c50dc0c6e13c</font>'
</pre>


修改存檔後，再次執行 brew install lftp ，即可完成安裝。

由於 Homebrew 是免費的開放源碼軟體，諸如此類的問題未來還會不斷發生，但是它至今仍有不斷在更新及維護，只要有更多人參與使用或開發，就會協助 Homebrew 變得更好。

您可以利用以下的管道，取得更多 Homebrew 的消息，並參與討論及提供建議。

* IRC (irc://irc.freenode.net/#machomebrew)
* Mailing List (homebrew@librelist.com)
* Twitter (http://twitter.com/machomebrew)
* GitHub (http://github.com/mxcl/homebrew)

### 延伸閱讀

* [Mac OS X 必備，好用的 Homebrew Cask 軟體安裝工具](http://blog.lyhdev.com/2014/10/mac-os-x-homebrew-cask.html)
* [Install Homebrew — The missing package manager for OS X](http://tsai-cookie.blogspot.tw/2015/10/install-homebrew-missing-package.html)
