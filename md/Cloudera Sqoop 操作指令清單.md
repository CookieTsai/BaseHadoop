# Cloudera Sqoop 操作指令清單

## 下載 JDBC

[下載連結](http://www.microsoft.com/zh-tw/download/details.aspx?id=11774)

## 解壓縮 tar.gz 檔案

	hostname:~ UserName$ tar -zxvf {File Name}

## 匯入 JDBC 

	hostname:~ UserName$ sudo cp {Unzip Directory}/sqljdbc4.jar /var/lib/sqoop

## Sqoop Import 指令

	hostname:~ UserName$ sqoop import --connect "jdbc:sqlserver://{SQL Server IP}:1433;databaseName={DataBase Name}" --username sa --password {Password} --table {Table Name} -m 1

## Sqoop Export 指令

	hostname:~ UserName$ sqoop export --connect "jdbc:sqlserver://{SQL Server IP}:1433;databaseName={DataBase Name}" --username sa --password {Password} --table {Table Name} -m 1 --export-dir {Export Directory} --input-fields-terminated-by ","

## 刪除檔案

	hostname:~ UserName$ hadoop fs -rm -r {資料夾名稱}




