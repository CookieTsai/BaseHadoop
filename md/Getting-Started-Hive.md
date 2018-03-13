<h1> Getting Started Hive </h1>

```
注意事項：
	1. 文章內容為僅供參考及學習使用
```

## Create Table

```
CREATE EXTERNAL TABLE OrderLog (
    date STRING,
    uid STRING,
    pid STRING,
    cnt INT,
    price INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/transaction';
```

## Select Table

```
SELECT * FROM OrderLog;
```

