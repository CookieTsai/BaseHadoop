<h1>淺談 Extract-Transform-Load (ETL)</h1>

```
注意事項：
	1. 文章內容主要以 Web Log 的資料轉換過程為主要案例，
	2. 文章內容僅作參考
```

## 目錄

[TOC]

## 前言

網頁日誌（Web Log）是一種常見的資料類型，通常可以透過分析日誌來了解，產品或是伺服器目前的現況，甚至是提供產品的未來走向。一般情況下網頁日誌中會存在許多雜訊，這些內容不一定是我們所需要的，因此，在使用時我們需要撰寫一些程式進行轉換。本篇文章內容著重於轉換過程的實現，文章僅提供單一種實現方式，再多數情況是需要依照現有專案進行調整。

## ETL

* Extract — 資料擷取：從資料來源處擷取所需之數據資料。
* Transform — 資料轉換：針對所擷取出之數據資料進行轉換。
* Load — 資料載入：將轉換後的資料載入到目的地。

<h2> Data Set </h2>

以下內容為網頁日誌，目的是作為本篇文章所使用的假資料集。（僅供練習使用）

<pre style="height: 200px; word-break: break-all; white-space: pre;">
1.172.0.185 - - [01/Jun/2011:00:00:00 +0800] "GET /action?;act=view;user=;uuid=6c27cc0b-d35a-addb-feb2-561b6bb28a5;pid=0006604986; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Linux; U; Android 4.1.1; zh-tw; PadFone 2 Build/JRO03L) AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Mobile Safari/534.30"
61.58.145.131 - - [01/Jun/2011:00:00:00 +0800] "GET /action?;act=view;user=;uuid=280f45c-dccf-fb70-a3a7-1f8f4ee7661a;pid=0024134891; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Linux; U; Android 4.1.2; zh-tw; GT-P3100 Build/JZO54K) AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Safari/534.30"
114.41.4.218 - - [01/Jun/2011:00:00:01 +0800] "GET /action?;act=order;user=U312622727;uuid=252b97f1-25bd-39ea-6006-3f3ebf52c80;buy=0006944501,1,1069; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0; MAARJS)"
114.43.89.52 - - [01/Jun/2011:00:00:01 +0800] "GET /action?;act=view;user=;uuid=7ec350c-3f5d-7e83-f03b-da5e92c148db;pid=0022226912; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)"
114.34.254.21 - - [01/Jun/2011:00:00:01 +0800] "GET /action?;act=view;user=;uuid=1f56717c-2be-6b6a-554c-b1c93634103a;pid=0023531314; HTTP/1.1" 302 160 "-" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; GTB7.5; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; eSobiSubscriber 2.0.4.16)"
111.184.100.252 - - [01/Jun/2011:00:00:03 +0800] "GET /action?;act=view;user=;uuid=e8caf452-433b-ab1f-976e-aa6b25799823;pid=0003848832; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
218.166.192.18 - - [01/Jun/2011:00:00:03 +0800] "GET /action?;act=view;user=U321001337;uuid=cbcaa5a5-4587-abac-b2f3-e2f2db2683eb;pid=0018926456; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
220.137.3.34 - - [01/Jun/2011:00:00:03 +0800] "GET /action?;act=order;user=U239012343;uuid=92e720da-17be-2b67-3383-9e5ccbd9499f;buy=0006018073,1,1680; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
66.249.80.72 - - [01/Jun/2011:00:00:04 +0800] "GET /action?;act=view;user=;uuid=d4a92f54-2fc7-a175-6c81-9d681cfe642e;pid=0018703860; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.4 (KHTML, like Gecko; Google Web Preview) Chrome/22.0.1229 Safari/537.4"
111.246.75.202 - - [01/Jun/2011:00:00:04 +0800] "GET /action?;act=view;user=;uuid=34bd97b0-49a1-774d-f8a3-ba30cb641173;pid=0022007845; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
114.46.250.60 - - [01/Jun/2011:00:00:18 +0800] "GET /action?;act=view;user=;uuid=a80c3ec-e9e0-9da3-591e-bbbb922de8;pid=0022008265; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (iPhone; CPU iPhone OS 6_1 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Mobile/10B143"
114.35.30.119 - - [01/Jun/2011:00:00:18 +0800] "GET /action?;act=view;user=;uuid=2d9850ec-dd2-b4d3-7af7-6a68d6297723;pid=0001891400; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
114.42.124.225 - - [01/Jun/2011:00:00:19 +0800] "GET /action?;act=order;user=U10007697373;uuid=81b98199-a680-f862-e21-991ba747318d;buy=0002267974,1,285; HTTP/1.1" 302 160 "-" "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; Trident/6.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; InfoPath.3; .NET4.0C; AskTbAVR-4/5.15.20.37949)"
59.126.110.102 - - [01/Jun/2011:00:00:19 +0800] "GET /action?;act=view;user=;uuid=73ad381-5de-645-e994-9e432402d967;pid=0004405251; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:22.0) Gecko/20100101 Firefox/22.0"
61.70.205.58 - - [01/Jun/2011:00:00:19 +0800] "GET /action?;act=view;user=;uuid=96823bb7-917b-85e0-3b12-8b55188c71c2;pid=0006402675; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
59.126.110.102 - - [01/Jun/2011:00:01:09 +0800] "GET /action?;act=view;user=;uuid=73ad381-5de-645-e994-9e432402d967;pid=0004862454; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:22.0) Gecko/20100101 Firefox/22.0"
111.249.169.26 - - [01/Jun/2011:00:01:09 +0800] "GET /action?;act=view;user=;uuid=9e2d9243-be4-6942-e17a-90ba2b59eadb;pid=0007258064; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
38.99.122.214 - - [01/Jun/2011:00:01:10 +0800] "GET /action?;act=order;user=U296328517;uuid=69ec3e27-579f-bbb-5572-7a173429f0a;buy=0016144236,1,550; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (X11; U; Linux x86_64; zh-tw) AppleWebKit/534.35 (KHTML, like Gecko)  Chrome/11.0.696.65 Safari/534.35 Puffin/2.9909AP"
122.121.32.128 - - [01/Jun/2011:00:01:11 +0800] "GET /action?;act=view;user=U465077047;uuid=990dedcc-9c10-b3c3-b9b8-42b424cbe05;pid=0011166271; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
118.169.219.170 - - [01/Jun/2011:00:01:11 +0800] "GET /action?;act=view;user=;uuid=88c750fd-dcb3-53f6-ed8d-be6e1b37b124;pid=0013923416; HTTP/1.1" 302 160 "-" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; Foxy/1; GTB7.5; Foxy/1; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; InfoPath.1; .NET4.0C; .NET CLR 1.1.4322; BRI/2)"
123.241.242.61 - - [01/Jun/2011:00:01:35 +0800] "GET /action?;act=view;user=;uuid=9f97cd65-319e-79eb-134b-42132b9ad900;pid=0022827254; HTTP/1.1" 302 160 "-" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; InfoPath.1; OfficeLiveConnector.1.3; OfficeLivePatch.0.0; yie8)"
125.230.67.137 - - [01/Jun/2011:00:01:35 +0800] "GET /action?;act=view;user=U46488849;uuid=d8d2d9f6-52d8-cbb5-2aef-baafd1472995;pid=0006842253; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.57 Safari/537.36"
182.234.93.248 - - [01/Jun/2011:00:01:36 +0800] "GET /action?;act=order;user=U300884570;uuid=97f37967-136a-4422-2776-25bb97c1477a;buy=0014516980122,1,249; HTTP/1.1" 302 160 "-" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0)"
123.194.77.142 - - [01/Jun/2011:00:01:36 +0800] "GET /action?;act=view;user=;uuid=b96731e6-5d2f-5322-bcfc-1734e4264441;pid=0002457490; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; NP08; MAAU; NP08)"
1.172.70.194 - - [01/Jun/2011:00:01:37 +0800] "GET /action?;act=view;user=;uuid=f65099ea-87a0-1a-e7c2-2a9dc5d8f3db;pid=0022772735; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.0; Trident/5.0)"
115.43.52.88 - - [01/Jun/2011:00:01:48 +0800] "GET /action?;act=view;user=;uuid=97cf13d1-2cca-ba6-196c-2aedcb3ffaf4;pid=0000248054; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)"
114.45.248.34 - - [01/Jun/2011:00:01:48 +0800] "GET /action?;act=view;user=;uuid=7cb94b4e-d7cc-423c-d0ee-cc33a55b325a;pid=0023531314; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
59.104.150.64 - - [01/Jun/2011:00:01:48 +0800] "GET /action?;act=order;user=U451050374;uuid=b7c64376-ae33-6b0-3008-7ebe4fdc656b;buy=0004134266,1,1780; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
36.239.20.203 - - [01/Jun/2011:00:01:48 +0800] "GET /action?;act=search;user=U238612347;uuid=27e6f25e-3acf-5951-3dd-dd7350e3aacb; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)"
175.182.5.59 - - [01/Jun/2011:00:01:48 +0800] "GET /action?;act=view;user=;uuid=4542e80f-c538-e6df-90b7-cd4bb565aa3b;pid=0006437874; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Linux; U; Android 4.0.3; zh-tw; Transformer TF101G Build/IML74K) AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Safari/534.30"
112.105.111.162 - - [01/Jun/2011:00:01:57 +0800] "GET /action?;act=view;user=;uuid=d97e1a57-1077-9a94-f3f1-862cbd642e7f;pid=0014476265; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)"
1.175.236.63 - - [01/Jun/2011:00:01:57 +0800] "GET /action?;act=view;user=;uuid=d2a968c-977f-75fb-382d-c98ca0b6460;pid=0019449102; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Linux; U; Android 2.3.5; zh-tw; HTC_DesireHD_A9191 Build/GRJ90) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1"
180.176.44.148 - - [01/Jun/2011:00:01:58 +0800] "GET /action?;act=order;user=U465124055;uuid=2c721756-52e8-ac1a-df48-a90ea2e45c25;buy=0012662252,1,488; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
59.126.110.102 - - [01/Jun/2011:00:01:58 +0800] "GET /action?;act=view;user=;uuid=73ad381-5de-645-e994-9e432402d967;pid=0006437863; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:22.0) Gecko/20100101 Firefox/22.0"
123.205.155.39 - - [01/Jun/2011:00:01:58 +0800] "GET /action?;act=view;user=;uuid=7d0c2051-c6b9-dfa-4a63-806e67339460;pid=0007398020; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
220.134.146.147 - - [01/Jun/2011:00:02:18 +0800] "GET /action?;act=view;user=U39528038;uuid=3f864dcd-aa7d-ce64-acfc-52e682c71ffe;pid=0002505510; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.57 Safari/537.36"
36.227.194.82 - - [01/Jun/2011:00:02:18 +0800] "GET /action?;act=view;user=;uuid=5238ae2d-c56e-8905-c9a6-aedb11dc3097;pid=0018692085; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8) AppleWebKit/534.59.8 (KHTML, like Gecko) Version/5.1.9 Safari/534.59.8"
111.249.190.187 - - [01/Jun/2011:00:02:18 +0800] "GET /action?;act=order;user=U403001364;uuid=b6a5f77f-2baa-d2be-13e0-ad1bb043169;buy=0004401294,1,276; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
112.104.126.65 - - [01/Jun/2011:00:02:19 +0800] "GET /action?;act=view;user=;uuid=f1098ee9-6afa-6d5a-c604-5e52d1b1eb8d;pid=0022657585; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
59.126.110.102 - - [01/Jun/2011:00:02:19 +0800] "GET /action?;act=view;user=;uuid=73ad381-5de-645-e994-9e432402d967;pid=0004405251; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:22.0) Gecko/20100101 Firefox/22.0"
59.126.110.102 - - [01/Jun/2011:00:03:11 +0800] "GET /action?;act=view;user=;uuid=73ad381-5de-645-e994-9e432402d967;pid=0004862454; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:22.0) Gecko/20100101 Firefox/22.0"
59.127.49.69 - - [01/Jun/2011:00:03:11 +0800] "GET /action?;act=view;user=;uuid=33512c49-50b0-235d-1235-f07e7a83113f;pid=0022827254; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
1.169.254.208 - - [01/Jun/2011:00:03:12 +0800] "GET /action?;act=order;user=U465123247;uuid=1d49d8b2-eb7c-fce3-ef73-73a3d2ff7d1d;buy=0000319874,1,2199; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.57 Safari/537.36"
121.254.111.211 - - [01/Jun/2011:00:03:12 +0800] "GET /action?;act=view;user=;uuid=a376c8c5-e5a0-3723-6a13-3633eb4229fd;pid=0011183491; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0; NP06)"
220.134.9.192 - - [01/Jun/2011:00:03:12 +0800] "GET /action?;act=view;user=;uuid=9a4fdb50-4adb-5f8a-f9d4-edd89a85b6b8;pid=0018335671; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Linux; U; Android 4.0.4; zh-tw; Enjoy 71 Build/IMM76I) AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Safari/534.30"
36.230.144.245 - - [01/Jun/2011:00:03:44 +0800] "GET /action?;act=view;user=;uuid=4aaeaca2-6d9b-dce-3cb7-beab21b171d9;pid=0013818475; HTTP/1.1" 302 160 "--3E" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)"
1.173.114.187 - - [01/Jun/2011:00:03:44 +0800] "GET /action?;act=view;user=;uuid=ccf35d73-ec1c-c745-dc38-acef82d0d6e1;pid=0022827254; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)"
114.41.4.218 - - [01/Jun/2011:00:03:45 +0800] "GET /action?;act=order;user=U311808547;uuid=252b97f1-25bd-39ea-6006-3f3ebf52c80;buy=0006944501,1,1069; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0; MAARJS)"
59.126.110.102 - - [01/Jun/2011:00:03:45 +0800] "GET /action?;act=view;user=;uuid=73ad381-5de-645-e994-9e432402d967;pid=0006437911; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:22.0) Gecko/20100101 Firefox/22.0"
111.249.186.158 - - [01/Jun/2011:00:03:45 +0800] "GET /action?;act=view;user=;uuid=293ad664-1e0b-fc36-132b-f254b56fd49f;pid=0003537671; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (iPad; CPU OS 6_1_3 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/6.0 Mobile/10B329 Safari/8536.25"
117.56.247.109 - - [01/Jun/2011:00:04:39 +0800] "GET /action?;act=view;user=;uuid=1647c87d-fc32-1ffa-db81-f3905c14b3bf;pid=0018585254; HTTP/1.1" 302 160 "-" "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Trident/4.0; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)"
59.126.110.102 - - [01/Jun/2011:00:04:40 +0800] "GET /action?;act=view;user=;uuid=73ad381-5de-645-e994-9e432402d967;pid=0004862454; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:22.0) Gecko/20100101 Firefox/22.0"
118.165.10.71 - - [01/Jun/2011:00:04:40 +0800] "GET /action?;act=order;user=U178630660;uuid=2bccfdfb-1c15-df86-9b27-61e0b5816931;buy=0003248055,1,799; HTTP/1.1" 302 160 "-" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.04506.648; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; Zune 4.7)"
36.239.110.197 - - [01/Jun/2011:00:04:40 +0800] "GET /action?;act=view;user=;uuid=89e920ac-aae-4b5-96ce-51c6433bb8;pid=0023620461; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0)"
61.70.164.55 - - [01/Jun/2011:00:04:40 +0800] "GET /action?;act=view;user=;uuid=e15e774c-b7e3-5b4d-104f-cb667875cdc;pid=0004931076; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
27.105.27.5 - - [01/Jun/2011:00:05:02 +0800] "GET /action?;act=view;user=;uuid=ee57807e-7ac7-cf3f-2242-3f1916e5c519;pid=0005552761; HTTP/1.1" 302 160 "--1A" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)"
118.171.251.122 - - [01/Jun/2011:00:05:03 +0800] "GET /action?;act=view;user=;uuid=1825275b-2cc3-7551-186a-cf8b370eb213;pid=0011092362; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (iPad; CPU OS 6_1_3 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/6.0 Mobile/10B329 Safari/8536.25"
114.32.85.222 - - [01/Jun/2011:00:05:03 +0800] "GET /action?;act=order;user=U339736346;uuid=1e66983f-f71c-b6ad-2b3f-a2a95ee25c5b;buy=0018924850,2,792,0000445944,1,404; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0; yie10)"
125.230.67.137 - - [01/Jun/2011:00:05:03 +0800] "GET /action?;act=view;user=U46488849;uuid=d8d2d9f6-52d8-cbb5-2aef-baafd1472995;pid=0005246791; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.57 Safari/537.36"
203.145.207.188 - - [01/Jun/2011:00:05:04 +0800] "GET /action?;act=view;user=;uuid=41ee27d6-5f83-b982-69f9-f378dc9fc11b;pid=0006437885; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:23.0) Gecko/20100101 Firefox/23.0"
36.227.194.82 - - [01/Jun/2011:00:05:24 +0800] "GET /action?;act=view;user=;uuid=5238ae2d-c56e-8905-c9a6-aedb11dc3097;pid=0009702873; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8) AppleWebKit/534.59.8 (KHTML, like Gecko) Version/5.1.9 Safari/534.59.8"
175.181.111.196 - - [01/Jun/2011:00:05:25 +0800] "GET /action?;act=view;user=;uuid=d252a0eb-e47c-f173-6f1c-33b4a36dad8e;pid=0000895123; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)"
219.71.161.82 - - [01/Jun/2011:00:05:25 +0800] "GET /action?;act=order;user=U465125772;uuid=5136439-b40a-997c-65ee-8100c0275bc2;buy=0013850723,1,2980; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
114.42.155.197 - - [01/Jun/2011:00:05:27 +0800] "GET /action?;act=view;user=;uuid=c8bdd4e9-3eb0-efa8-210f-1eeb716df30;pid=0018906005; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0)"
59.126.110.102 - - [01/Jun/2011:00:05:27 +0800] "GET /action?;act=view;user=;uuid=73ad381-5de-645-e994-9e432402d967;pid=0018307096; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:22.0) Gecko/20100101 Firefox/22.0"
123.240.63.53 - - [01/Jun/2011:00:05:29 +0800] "GET /action?;act=view;user=;uuid=f187192f-bfdb-35c1-6b10-9446e2aefd;pid=0022529850; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.0; Trident/5.0; yie9)"
124.10.91.92 - - [01/Jun/2011:00:05:30 +0800] "GET /action?;act=view;user=U396120416;uuid=3560692f-8134-a93e-8de3-f9197d0d500d;pid=0002940055; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0; MATPJS)"
123.241.44.19 - - [01/Jun/2011:00:05:31 +0800] "GET /action?;act=order;user=U465125671;uuid=b37521a3-de6c-193b-6b1-e2d1ad25205a;buy=0003448524,2,390; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
123.194.132.113 - - [01/Jun/2011:00:05:31 +0800] "GET /action?;act=view;user=;uuid=fe75b088-8c7c-44bc-31a9-e6bba557db64;pid=0024036390; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0; yie10)"
1.164.172.173 - - [01/Jun/2011:00:05:31 +0800] "GET /action?;act=view;user=;uuid=3168c1e7-ccfe-3e33-8393-b924b686d71c;pid=0023719010; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)"
203.67.201.249 - - [01/Jun/2011:00:08:05 +0800] "GET /action?;act=view;user=;uuid=70eb8b32-5e7-c1e5-f50a-8618e2338d44;pid=0006437874; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
114.45.171.181 - - [01/Jun/2011:00:08:05 +0800] "GET /action?;act=view;user=;uuid=158e8a19-bcd2-6468-3776-ce0abc30a533;pid=0011625036; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.57 Safari/537.36"
112.104.18.47 - - [01/Jun/2011:00:08:06 +0800] "GET /action?;act=order;user=U46498056;uuid=59f2aa24-24bc-3ead-59bd-4344c6b84d41;buy=0002689481,1,702; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
112.105.80.242 - - [01/Jun/2011:00:08:06 +0800] "GET /action?;act=view;user=;uuid=715a2c8-95eb-c074-4d7-52e718336da0;pid=0001854930; HTTP/1.1" 302 160 "--1A" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
203.74.127.204 - - [01/Jun/2011:00:08:06 +0800] "GET /action?;act=view;user=;uuid=a1dd90b6-5b7f-82f-9bb9-439e5c4ae5d3;pid=0020225225; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.57 Safari/537.36"
114.44.29.194 - - [01/Jun/2011:00:08:28 +0800] "GET /action?;act=view;user=;uuid=9582bf27-6b8e-777f-6ff5-d516dd535cdc;pid=0018746066; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
59.127.202.153 - - [01/Jun/2011:00:08:28 +0800] "GET /action?;act=view;user=;uuid=f3d5e4b3-420d-487a-6c34-9f4f9cbb27fd;pid=0014469055; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
1.162.213.10 - - [01/Jun/2011:00:08:28 +0800] "GET /action?;act=order;user=U349686362;uuid=5b8d9215-7f1c-596e-eee7-60b55d4ae3e0;buy=0005667605,1,488; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
219.68.8.186 - - [01/Jun/2011:00:08:28 +0800] "GET /action?;act=view;user=;uuid=7734290-d9ac-6bbb-9d4-15ffb4371b;pid=0003327295; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)"
123.50.54.134 - - [01/Jun/2011:00:08:28 +0800] "GET /action?;act=view;user=;uuid=22e38b54-c43f-d0a9-8f9f-9dd48eef476f;pid=0014622650; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
203.70.12.83 - - [01/Jun/2011:00:09:30 +0800] "GET /action?;act=view;user=;uuid=882097e0-9f65-f51b-163c-378faa8c1a8f;pid=0012998370; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.57 Safari/537.36"
114.40.13.94 - - [01/Jun/2011:00:09:31 +0800] "GET /action?;act=view;user=U384948322;uuid=3e709ba4-329c-7b9f-3591-65b9653728f6;pid=0018585254; HTTP/1.1" 302 160 "-" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0; GTB7.5; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; InfoPath.2)"
220.137.3.34 - - [01/Jun/2011:00:09:31 +0800] "GET /action?;act=order;user=U234579365;uuid=92e720da-17be-2b67-3383-9e5ccbd9499f;buy=0024026973,1,198; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
1.161.156.3 - - [01/Jun/2011:00:09:32 +0800] "GET /action?;act=view;user=;uuid=18a0d6df-7886-e8b5-5620-9b0722f968bd;pid=0023395282; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
123.50.54.134 - - [01/Jun/2011:00:09:32 +0800] "GET /action?;act=view;user=;uuid=22e38b54-c43f-d0a9-8f9f-9dd48eef476f;pid=0001756355; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
66.249.80.72 - - [01/Jun/2011:00:10:04 +0800] "GET /action?;act=view;user=;uuid=12c8ef3-a8a6-8beb-18ff-fde03d55559d;pid=0018523260; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.4 (KHTML, like Gecko; Google Web Preview) Chrome/22.0.1229 Safari/537.4"
59.126.110.102 - - [01/Jun/2011:00:10:04 +0800] "GET /action?;act=view;user=;uuid=73ad381-5de-645-e994-9e432402d967;pid=0004906226; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:22.0) Gecko/20100101 Firefox/22.0"
220.133.231.97 - - [01/Jun/2011:00:10:04 +0800] "GET /action?;act=order;user=;uuid=cf801dc4-c447-d607-e73b-583eafeba97e;buy=; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:23.0) Gecko/20100101 Firefox/23.0"
125.230.67.137 - - [01/Jun/2011:00:10:05 +0800] "GET /action?;act=view;user=U46488849;uuid=d8d2d9f6-52d8-cbb5-2aef-baafd1472995;pid=0018758865; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.57 Safari/537.36"
219.68.11.162 - - [01/Jun/2011:00:10:06 +0800] "GET /action?;act=view;user=;uuid=aec31212-ed7-ec04-92dc-8632b3e8c82d;pid=0018381661; HTTP/1.1" 302 160 "--2" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0; GTB7.5; Foxy/1; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C)"
123.204.128.45 - - [01/Jun/2011:00:10:18 +0800] "GET /action?;act=view;user=;uuid=cb0be90a-adcb-2145-74aa-fc4efe7a3cee;pid=0023720970; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (iPad; U; CPU OS 4_3_5 like Mac OS X; zh-tw) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8L1 Safari/6533.18.5"
203.145.207.188 - - [01/Jun/2011:00:10:18 +0800] "GET /action?;act=view;user=;uuid=41ee27d6-5f83-b982-69f9-f378dc9fc11b;pid=0018316266; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:23.0) Gecko/20100101 Firefox/23.0"
220.142.124.5 - - [01/Jun/2011:00:10:18 +0800] "GET /action?;act=order;user=U438707274;uuid=2ee31476-d557-d19a-31c7-3c2ce8f67d4;buy=0023733533,1,1710,0000445966,1,808; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
114.34.148.192 - - [01/Jun/2011:00:10:19 +0800] "GET /action?;act=view;user=U395982321;uuid=493ed0f-54fd-319b-2df9-bdb9d6f21c9c;pid=0018435454; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Linux; U; Android 4.1.1; zh-tw; HTC_One_X Build/JRO03C) AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Mobile Safari/534.30"
61.58.178.150 - - [01/Jun/2011:00:10:19 +0800] "GET /action?;act=view;user=;uuid=c4c3c8dc-913a-a218-4dc4-80d442d2d0ce;pid=0006587254; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Linux; U; Android 4.0.3; zh-tw; GT-I9100 Build/IML74K) AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Mobile Safari/534.30"
203.145.207.188 - - [01/Jun/2011:00:10:37 +0800] "GET /action?;act=view;user=;uuid=41ee27d6-5f83-b982-69f9-f378dc9fc11b;pid=0013038863; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:23.0) Gecko/20100101 Firefox/23.0"
1.162.69.186 - - [01/Jun/2011:00:10:37 +0800] "GET /action?;act=view;user=U126324055;uuid=f0d2caa8-aa16-9af9-cca2-c1f25b5de855;pid=0006437874; HTTP/1.1" 302 160 "-" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; InfoPath.1; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; YTB730; .NET4.0C; .NET4.0E)"
220.142.124.5 - - [01/Jun/2011:00:10:37 +0800] "GET /action?;act=order;user=U438707274;uuid=2ee31476-d557-d19a-31c7-3c2ce8f67d4;buy=0023733533,1,1710,0000445966,1,808; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
1.34.131.167 - - [01/Jun/2011:00:10:37 +0800] "GET /action?;act=search;user=;uuid=dc945994-2472-2cf5-2fdc-eb85defc5465; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.62 Safari/537.36"
219.85.82.234 - - [01/Jun/2011:00:10:38 +0800] "GET /action?;act=search;user=;uuid=3357b74b-2e20-7b63-ccd7-6b5613a446c9; HTTP/1.1" 302 160 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.95 Safari/537.36"
</pre>


<h2> Training </h2>

<h3> 目標一：Log to CSV </h3>

尚未整理過的 Log 對於任何處理都是不方便的，在很多時候甚至會是效能的瓶頸。所以第一步可以選用常用的格式如：csv, tsv...等，先將要使用的資料進行統一的轉換。

<h4> Sample Code：log2csv.py </h4>

```
import sys
import re
import csv

index = {"ip":0,"day":1,"time":2,"act":3,"user":4,"uuid":5,"buy":6}

writer = csv.writer(sys.stdout)

for line in sys.stdin:
    row=["NA","NA","NA","NA","NA","NA","NA"]

    line = re.sub(r'"\s.*$', '', line)
    m = re.search(r'^([0-9.]+).*\[([^:]+):(\S+).*\?;(\S+)', line)

    row[index["ip"]] = m.group(1)
    row[index["day"]] = m.group(2)
    row[index["time"]] = m.group(3)

    for item in re.findall(r'(\w+)=([^;]+)', m.group(4)):
        key = item[0].strip()
        val = item[1].strip()
        if key in index:
            row[index[key]] = val

    writer.writerow(row)
```

<h3> 目標二：CSV to Order's CSV </h3>

本文此次的目標以取出 Web Log 中 act=order 類型的資料為範例。由於 order 的 buy 資料內容為（商品代碼,購買數量,購買金額... 反覆出現），與一般資料如：user 的內容不同，因此，在處理 order 時需要將重複出現的內容再切割成數份。撰寫 csv2order.py 來解決以上問題，詳細做法請看程式碼。

<h4> Sample Code：csv2order.py </h4>

```
import sys
import csv
import re

index = {"day":1,"user":4,"buy":6}

reader = csv.reader(sys.stdin)
writer = csv.writer(sys.stdout)

for row in reader:
    out = ["NA","NA","NA","NA","NA"]
    out[0] = row[index["day"]]
    out[1] = row[index["user"]]

    for item in re.findall(r'([0-9]+),([0-9]+),([0-9]+)', row[index["buy"]]):
        out[2] = item[0]
        out[3] = item[1]
        out[4] = item[2]
        writer.writerow(out)
```

## 執行

<h3> 指令 </h3>

```
$ cat {網頁日誌 路徑} | python log2csv.py | python  csv2order.py
```

<h3> 輸出 </h3>

```
01/Jun/2011,U312622727,0006944501,1,1069
01/Jun/2011,U239012343,0006018073,1,1680
01/Jun/2011,U10007697373,0002267974,1,285
01/Jun/2011,U296328517,0016144236,1,550
01/Jun/2011,U300884570,0014516980122,1,249
01/Jun/2011,U451050374,0004134266,1,1780
01/Jun/2011,U465124055,0012662252,1,488
01/Jun/2011,U403001364,0004401294,1,276
01/Jun/2011,U465123247,0000319874,1,2199
01/Jun/2011,U311808547,0006944501,1,1069
01/Jun/2011,U178630660,0003248055,1,799
01/Jun/2011,U339736346,0018924850,2,792
01/Jun/2011,U339736346,0000445944,1,404
01/Jun/2011,U465125772,0013850723,1,2980
01/Jun/2011,U465125671,0003448524,2,390
01/Jun/2011,U46498056,0002689481,1,702
01/Jun/2011,U349686362,0005667605,1,488
01/Jun/2011,U234579365,0024026973,1,198
01/Jun/2011,U438707274,0023733533,1,1710
01/Jun/2011,U438707274,0000445966,1,808
01/Jun/2011,U438707274,0023733533,1,1710
01/Jun/2011,U438707274,0000445966,1,808
```

## 後記

<h3> 與 Hadoop Hive 結合 </h3>

* 以下操作皆在 Localhost 下進行操作
* 目標為上傳資料至 HDFS 並且使用 Hive 建立相對應表格後查詢 

<h4> 上傳至 HDFS </h4>

```
$ hadoop fs -mkdir /table
$ hadoop fs -put {order's csv 檔案路徑} /table
```

<h4> Using Beeline </h4>

```
$ beeline -u jdbc:hive2://localhost:10000
```

<h4> Create Table </h4>

```
CREATE EXTERNAL TABLE OrderLog (
    d STRING,
    uid STRING,
    pid STRING,
    cnt INT,
    price INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/table';
```

<h4> Select Table </h4>


```
SELECT * FROM OrderLog;
```




