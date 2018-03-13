```
注意事項：
	1. 本文內容主要為小編自己的學習路徑分享
	2. 在使用 Telnet 時須注意換行符號，建議純手動打勿複製貼上
	3. 文章中所使用到的網址可以任意更動
```

## Catalog

[TOC]

## Abstract

近日小編在工作上遇到許多連線問題，經過一段時間的摸索意外讓小編學會了使用 Socket 來發送 HTTP Request，而後逐步的學會了使用 `telnet` 和 `s_client` 等工具來模擬長連結的 HTTP Request，也認識了網路中的各種連線原來都是利用 Socket 實現，本篇文章將透過一個小測試，來讓各位認識 Web 是如何與 Server 進行溝通請求。

<h3> 關於 HTTP 的小知識 </h3>

扣除早期 0.X 版的 HTTP 目前現行的 HTTP Protocol 多為 1.0 版及 1.1 版，最新版本為 2.0 版非目前主流（本篇內容不使用 2.0 版本），在 1.0 與 1.1 最大的差異在於短連結與長連結，1.0 版預設為短連結（Connection 預設為 close 1.1 則反之）

一個標準的 HTTP Protocol 大致分成 Head 與 Body 兩個部分，中間使用一個換行來進行分隔，範例如下：

<pre>
<font color="red">POST</font> <font color="green">/2016/06/extract-transform-load.html</font> <font color="blue">HTTP/1.1</font>
<font color="orange">Host: tsai-cookie.blogspot.tw
User-Agent: User Defient/1.0
Content-Type: text/plain; charset=utf-8</font>

<font color="purple"><.... Your Data ....></font>

</pre>

<font color="red">紅：</font>Method<br />
<font color="green">綠：</font>Path<br />
<font color="blue">藍：</font>HTTP Version<br />
<font color="orange">橘：</font>Head<br />
<font color="purple">紫：</font>Body（許多地方文章稱為 Entity）<br />

P.S 在實驗中，此 POST HTTP Request 伺服器並不支援，僅供參考用。

並不是所有 HTTP Request 都有 body 最常見的四種 GET, POST, DELETE 和 PUT 中，只有 POST 和 PUT 可以攜帶 Body，而沒有攜帶 Body 的 Request 就不用 Content-Type 和 Body，反之則需要。

無 Body 的 Request，範例如下：

<pre>
<font color="red">GET</font> <font color="green">/2016/06/extract-transform-load.html</font> <font color="blue">HTTP/1.1</font>
<font color="orange">Host: tsai-cookie.blogspot.tw
User-Agent: User Defient/1.0</font>

</pre>

## Experiment

接下來小編將帶各位做一個簡單的實驗，利用 telnet 及 s_client 向 www.google.com.tw 發送一個手製的 HTTP Request，首先，我們需要一個完整的 HTTP Reuqest，內容如下：

```
GET / HTTP/1.1
Host: www.google.com.tw
User-Agent: CookieTsai
Connection: close


```

P.S `Connection: close` 目的為告訴 Server 做完後結束連線，如不要這行可以連續發送數次。

如您使用的作業系統為 Windows，想要做此實驗須先手動打開 telnet 或安裝 cygwin，本文未提供方法造成不便請見諒。

<h3> Telnet </h3>

於 `命令提示字元` 或 `終端機` 使用指令

```
$ telnet www.google.com.tw 80
Trying 64.233.187.94...
Connected to www.google.com.tw.
Escape character is '^]'.
```

進入後輸入 Request（紅色）並按下 Entry（按 1~2 次需產生一行空白行）即會收到 Server 回應（藍色）

內容如下：

<pre>
<font color="red">GET / HTTP/1.1
Host: www.google.com.tw
User-Agent: CookieTsai
Connection: close</font>

<font color="blue">HTTP/1.1 200 OK
Date: Wed, 27 Jul 2016 11:24:19 GMT
Expires: -1
Cache-Control: private, max-age=0
Content-Type: text/html; charset=Big5
P3P: CP="This is not a P3P policy! See https://www.google.com/support/accounts/answer/151657?hl=en for more info."
Server: gws
X-XSS-Protection: 1; mode=block
X-Frame-Options: SAMEORIGIN
Set-Cookie: NID=83=xuZXlGKYIFhYU7tlUD9YzxAcNuOxvrFSNa191FEEdOxB0s8xpW3W-u8t4BQnURrmtGuDVbgAYzq6Ed9MtCwqPdfGndeJ6GieOpNkBF5qaztXmm5H3o5CQ-FukayW1fhH; expires=Thu, 26-Jan-2017 11:24:19 GMT; path=/; domain=.google.com.tw; HttpOnly
Accept-Ranges: none
Vary: Accept-Encoding
Transfer-Encoding: chunked

28ba...（略）</font>

</pre>

<h3> Openssl's s_client </h3>

既然使用 telnet 就可以發送了，為何需要使用 openssl 的 s_client 呢？

許多網站為了資訊安全會使用 HTTPS，這並不是指有一種 Protocol 是 HTTPS，而是一般的 HTTP Protocol + SSL，另一種說法是 HTTP Protocol 在傳送時加上 SSL 的加密方法。這時候單純的 telnet 無法提供 SSL 加解密，因此，要利用 openssl 提供的 s_client 來發送請球，內容大致如下。

於 `命令提示字元` 或 `終端機` 使用指令

```
$ openssl s_client -connect www.google.com.tw:443
CONNECTED(00000003)
depth=2 /C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
verify error:num=20:unable to get local issuer certificate
verify return:0
---
Certificate chain
 0 s:/C=US/ST=California/L=Mountain View/O=Google Inc/CN=google.com
   i:/C=US/O=Google Inc/CN=Google Internet Authority G2
 1 s:/C=US/O=Google Inc/CN=Google Internet Authority G2
   i:/C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
 2 s:/C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
   i:/C=US/O=Equifax/OU=Equifax Secure Certificate Authority
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIgHDCCHwSgAwIBAgIIEAtlXh5wxiwwDQYJKoZIhvcNAQELBQAwSTELMAkGA1UE
BhMCVVMxEzARBgNVBAoTCkdvb2dsZSBJbmMxJTAjBgNVBAMTHEdvb2dsZSBJbnRl
cm5ldCBBdXRob3JpdHkgRzIwHhcNMTYwNzEzMTMxNjMyWhcNMTYxMDA1MTMxNjAw
...
...（中間省略非常長）...
...
Q57MUEhLPbQnNPaP5KXQfMB+9GbXrCGwJzX+Uue0T99zo/UuKW1pJUDM8jDhgMP3
Io5fc9LqwWuWDARjvODD6L7ZefN192ovI41+3yJ6ImOUvPJR31M4ZqiMT2MSJ1NT
k2w8H4X3z0pLvtlRFyDoq5TwkU8r6QnBQFD/rC6tAqvx7WGLc2nRXy2bgcKItwC/
uO9p+m2wqZJPPCQOnE0m6A==
-----END CERTIFICATE-----
subject=/C=US/ST=California/L=Mountain View/O=Google Inc/CN=google.com
issuer=/C=US/O=Google Inc/CN=Google Internet Authority G2
---
No client certificate CA names sent
---
SSL handshake has read 10308 bytes and written 456 bytes
---
New, TLSv1/SSLv3, Cipher is AES128-SHA
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1
    Cipher    : AES128-SHA
    Session-ID: 13F7455348699FFF98FB6D11865EDDAEB3F904D6AC727041CA4F3857A0E97226
    Session-ID-ctx:
    Master-Key: F74606075CAC3B709F39CA9F2C699FCEA892C16F5E58AF0EC453433F74B0E7F7FD10890168C19E8EEAB1727F5CD4A478
    Key-Arg   : None
    Start Time: 1469623117
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
```

進入後輸入 Request（紅色）並按下 Entry（按 1~2 次需產生一行空白行）即會收到 Server 回應（藍色）

內容如下：

<pre>
<font color="red">GET / HTTP/1.1
Host: www.google.com.tw
User-Agent: CookieTsai
Connection: close</font>

<font color="blue">HTTP/1.1 200 OK
Date: Wed, 27 Jul 2016 12:39:25 GMT
Expires: -1
Cache-Control: private, max-age=0
Content-Type: text/html; charset=Big5
P3P: CP="This is not a P3P policy! See https://www.google.com/support/accounts/answer/151657?hl=en for more info."
Server: gws
X-XSS-Protection: 1; mode=block
X-Frame-Options: SAMEORIGIN
Set-Cookie: NID=83=kdPJl42ANhiQji2fmjEO4h9aOjwp6ZWcBvXIREOnOVYBfWlS7PMjwn1w8ZTlc_gUjbXvQvnMGkJkKnsZGLy7CcDVO0s40Q7mofOTY4t2Uvt_kVSD7KumameS8PYDvLtj; expires=Thu, 26-Jan-2017 12:39:25 GMT; path=/; domain=.google.com.tw; HttpOnly
Alternate-Protocol: 443:quic
Alt-Svc: quic=":443"; ma=2592000; v="36,35,34,33,32,31,30,29,28,27,26,25"
Accept-Ranges: none
Vary: Accept-Encoding
Connection: close

...（略）</font>

</pre>



