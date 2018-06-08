## Install MariaDB + PHP + PhpMyAdmin on MacOS

```
注意事項：
    1. 文章內容為小編完工後憑印象撰寫，如有錯誤需自行排除
    2. 文章僅供參考
```

### 問題

某日小編想測試一下 Java Hibernate 連線 MariaDB 進行測試，順利告一個段落後，心想把 PhpMyAdmin 也安裝好方便使用，沒想到一路跌跌撞撞一堆問題，終於安裝完成特寫此篇文章以抒發心情。（謎之音：其實怕哪天又要裝寫文章記錄）

### 安裝 MariaDB

一開始小編不知道為什麼想去官網下載安裝包下來手動安裝，殊不知連續換三兩個版本都會在最後一步忽然告知安裝失敗，而且不寫原因... 最後安裝方式...

```
$ brew install mariadb
```

P.S. 請愛用 brew 真心好用...

### 安裝 PHP

在 MacOS 裡面其實有內建 `apachectl` 可以直接使用指令 `sudo apachectl start`，不過預設 PHP 沒有被引用需要手動開啟。

開啟方法如下：

* 步驟一： `vim` 編輯 `/etc/apache2/httpd.conf`
* 步驟二：找到 `#LoadModule php5_module libexec/apache2/libphp5.so` 將 `#` 移除
* 步驟三：除新啟動伺服器`sudo apachectl restart`

### 安裝 PhpMyAdmin

從官網上下載最新的 PhpMyAdmin 解壓縮後放入`/Library/WebServer/Documents/`，之後由此開始小編遇到一連串的 Permission Denied 跟一些怪怪問題。

* 問題一：session_start 時出現 Permission Denied (13)

解決方法：

```
$ echo '<? session_start(); ?>' > /Library/WebServer/Documents/session.php
```

使用網頁開啟`http://localhost/session.php`後會顯示 Denied 的路徑小編遇到的位置是 `/var/tmp`，將此路徑的權限更改為當前使用者即可排除。

* 問題二：使用登入後出現 `mysqli_real_connect(): (HY000/2002)`

按照網路上說法好像是 MacOS 在解析 config.inc.php 中的 localhost 會判斷錯誤，因此將此檔案中的值更改為 127.0.0.1 即可排除。

指令如下：

```
cd /Library/WebServer/Documents/{your phpmyadmin's folder name}
cp config.sample.inc.php config.inc.php
vim config.inc.php
```
修改 `$cfg['Servers'][$i]['host'] = 'localhost';` 的 localhost 改成 127.0.0.1

### 總結

破除各種麻煩後終於成功登入，這是小編從學生時代至現在第一次覺得 PhpMyAdmin 這麼難安裝，給跟小編一樣想不開在 MacOS 安裝伺服器的朋友們。




