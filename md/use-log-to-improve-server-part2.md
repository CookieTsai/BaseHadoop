<h1>從 Log 來進行伺服器的優化之 3 秒處理 2 億資料</h1>

<pre>
注意事項：
	1. 本文僅供參考
	2. 內容經過實際驗證及使用
	3. 電腦規格, 4 Num of CPUs, 8G Memory, 50G Storage 僅一台
</pre>

## 目錄

* [前言](#toc_1)
* [方法](#toc_2)
* [成果](#toc_3)
* [結語](#toc_4)

## 前言

自上一篇文章[從 Log 來進行伺服器的優化](http://tsai-cookie.blogspot.com/2018/03/use-log-to-improve-server.html)後，系統順利運行了一陣子，接著就遇到了一個巨大的困擾。當大量的 Log 放入到 Impala 後，任意使用一個 SQL 指令會需要運行約 <font color="#C22">5 ~ 6 分鐘甚至 10 分鐘以上</font>，此時的資料大約為 6 千萬筆資料。當期望透過這些資料產出一些報表時，通常會需要反覆的查詢資料，而單一次的執行時間就如此長，使得整個系統變得不堪使用。

為此，小編開始進行一連串的優化，在使用 `Gzip`, `Partition`, `Parquet` 及 `Compute Stats` 相關的技術後，系統成功達到比原先小量資料更快的效能，而且更加節省硬碟的空間。在本文中的[成果](#toc_3)看到，目前伺服器存在 2 億筆資料，而一個簡單的計數查詢可以在 3 秒左右完成，可以說在優化後有著天翻地覆的變化，文章中小編將分享這幾項技術，以及使用的差異與重點。

## 方法

<h3> Gzip + Partition + Parquet + Compute Stats </h3>

**Information**

* **Gzip**

	這是一種壓縮格式，Hive 跟 Impala 都有支援，可以新增使用 TXT 格式的 Table，載入 gzip 過的 TXT 文件，運作是正常的，但是由於 gzip 在 Hive 和 Impala 中不可切割，需注意單一檔案的資料大小。
	
	P.S. 小編使用此壓縮格式的原因是 Log 太大，而伺服器只有空間不足，如果空間充裕可以不使用。

* **Partition**

	一般來說如果未作優化的 Hive 會將資料完全存放於同一個資料夾中，當今天要找部分資料時，Hive 或 Impala 會將所有資料掃過所有資料，將有符合條件的資料取出。而使用了 Partition 的效果等於我們事先將資料分類好，我們可以使用 WHERE 的條件，只找特定資料夾所對應的內容。因此，在資料量極大時這樣只取需要的資料處理，就變得非常的重要。
	
* **Parquet**

	Apache Parquet is a [columnar storage](https://en.wikipedia.org/wiki/Column-oriented_DBMS) format available to any project in the Hadoop ecosystem, regardless of the choice of data processing framework, data model or programming language. Source From：[Apache Parquet官網](https://parquet.apache.org/)
	
	Apache Parquet 是一種使用 Column-oriented 方式存儲的資料格式，這一類型的資料格式非常適合用於查詢與統計運算。其最主要快速的原因為同一類型的資料會被存放在同一行裡面，在取得資料時不會去取得多餘的資料，以及這一類型的資料通常在存放時，會額外先算出每一行的基本資訊，如：筆數。因此，在實際運算時甚至不必去取出資料就可以算出筆數。

* **Compute Stats**

	這是一個 Impala 官方提供可以提升效率的一個指令，使用指令後 Impala 會去收集資料表中的一些統計數據數據, 資料分佈位置及各區塊的資料量... 等，幫助 Impala 在執行時可以有效的去運行整個流程，減少不必要的運算。

**Step Flow**

1. 建立一個 Table 使用 TXT 格式（In Hive）

		CREATE EXTERNAL TABLE TxtLog (
		  t TIMESTAMP,
		  ui STRING,
		  li STRING,
		  api STRING,
		  ua STRING,
		  rt INT,
		  st STRING
		)
		ROW FORMAT DELIMITED
		FIELDS TERMINATED BY ','
		STORED AS TEXTFILE;

2. 將 Gzip 資料匯入第一步所產生的 TXT Table（In Hive）

		LOAD DATA LOCAL INPATH '/home/cloudera/log.csv.gz' INTO TABLE TxtLog;

	註解：此指令中的路徑並非 HDFS 的路徑，而是當前伺服器的檔案系統。

3. 建立一個 Table 使用 Parquet 格式並且設定 Partition（In Hive）

		CREATE EXTERNAL TABLE PaPaLog (
		  t TIMESTAMP,
		  ui STRING,
		  li STRING,
		  api STRING,
		  ua STRING,
		  rt INT,
		  st STRING
		)
		PARTITIONED BY (m INT, d INT)
		STORED AS PARQUET;

	註解：這裡的 Partition 是使用 Month 與 Day 做分類，可依需求自行調整

4. Impala 刷新同步 Hive's Metadata（In Impala）

		INVALIDATE METADATA;
		
	註解：Impala 可以通過 REFRESH 和 INVALIDATE METADATA 進行同步，可依需求使用。

5. 從 TXT Table 寫入 Parquet Table（In Impala）

	* 全部匯入

			INSERT OVERWRITE TABLE PaPaLog PARTITION(m, d)
			SELECT *, month(t) as m, day(t) AS d FROM TxtLog;
		
	* 部分匯入

			INSERT OVERWRITE TABLE PaPaLog PARTITION(m=3, d)
			SELECT *, day(t) AS d FROM TxtLog WHERE month(t)=3;
	
	註解 1：此步驟要注意如果 TxtLog 資料過大有可能發生 OOM，請依需求調整成部分匯入 <br />
	註解 2：在這一步匯入時，小編會將 TxtLog 僅保留一個月的資料，這樣可以在匯入時會加快速度

6. 執行 Compute Stats（In Impala）

		COMPUTE STATS PaPaLog;
		
	註解：推薦執行，但在此案例中`COMPUTE STATS`做之後的速度差異性很小

## 成果

```
Starting Impala Shell without Kerberos authentication
Connected to quickstart.cloudera:21000
Server version: impalad version 2.9.0-cdh5.12.0 RELEASE (build 03c6ddbdcec39238be4f5b14a300d5c4f576097e)
***********************************************************************************
Welcome to the Impala shell.
(Impala Shell v2.9.0-cdh5.12.0 (03c6ddb) built on Thu Jun 29 04:17:31 PDT 2017)

You can change the Impala daemon that you're connected to by using the CONNECT
command.To see how Impala will plan to run your query without actually executing
it, use the EXPLAIN command. You can change the level of detail in the EXPLAIN
output by setting the EXPLAIN_LEVEL query option.
***********************************************************************************
[quickstart.cloudera:21000] > SELECT COUNT(*) FROM PaPaLog;
Query: select COUNT(*) FROM PaPaLog
Query submitted at: 2018-06-07 10:17:17 (Coordinator: http://quickstart.cloudera:25000)
Query progress can be monitored at: 
http://quickstart.cloudera:25000/query_plan?query_id=bb44c4b554854e7d:766d66a000000000
+-----------+
| count(*)  |
+-----------+
| 212103093 |
+-----------+
Fetched 1 row(s) in 3.03s
```

## 結語

除了以上的方法，Hive 及 Impala 還有其他方式可以提升效能，小編只使用了一部分方法，也是一些比較主流的方法。能夠真正有效加快效能的永遠是適當的格式，以及只取必要的資料，還有適當的演算方式。因此，如果文章無法滿足需求，小編建議可以優先往如何只取必要性資料當方向。

