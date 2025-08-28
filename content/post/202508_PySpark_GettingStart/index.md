---
title: PySpark 學習筆記 - Getting Start
description: Getting Start 快速瀏覽一遍 PySpark 開發用法
slug: pyspark_getting_start
date: 2025-08-29 00:00:00+0000
image: 
categories:
    - PySpark
    - Data Engineering
tags:
    - PySpark
    - Data Engineering
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

# 🔥 PySpark - Getting Start 

PySpark 學習

本筆記涵蓋以下內容：

- 建立 Spark Session
- 讀取 CSV 原始資料
- 欄位標準化（批次重新命名）
- 建立暫存表供 SQL 查詢
- 查詢與轉換範例：週次統計
---
Spark 官方文件是你個好夥伴  [spark.apache.org](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/core_classes.html)  
DataFrame Methods
1. Actions
Are DataFrame Operations that kick off a Spark Job execution and return to the Spark Driver
- collect
- count
- describe
- first
- foreach
- foreachPartition
- head
- show
- summary
- tail
- take
- toLocalIterator

2. Transformations
Spark DataFrame transformation produces a newly transformed Dataframe
- agg
- alias
- coalesce
- colRegex
- crossJoin
- crosstab
- cube
- distinct
- drop
- drop_duplicates
- dropDuplicates
- dropna
- exceptAll
- filter
- groupby
...

3. Functions/Methods
Dataframe Methods or function which are not categorized into Actions or Transformations
- approxQuantile
- cache
- checkpoint
- createGlobalTempView
- createOrReplaceGlobalTempView
- createOrReplaceTempView
- createTempView
- explain
- hint
- inputFiles
- isLocal
- localCheckpoint
- toDF
- toJSON
...
---

[Dataset - Fire Department](https://data.sfgov.org/Public-Safety/Fire-Department-and-Emergency-Medical-Services-Dis/nuek-vuh3/about_data)
## 📦 1. 建立 Spark Session 與讀取資料

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *

spark = SparkSession.builder \
    .appName("basic") \
    .master("local[2]") \
    .getOrCreate()

fire_df = spark.read \
    .format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("data/sf_file_calls.csv")
```

---

## 🛠️ 2. 欄位重新命名（標準化）

有些欄位包含空格或不適合程式使用的命名，透過字典映射統一名稱：

```python
rename_map = {
    "Call Number": "CallNumber",
    "Unit ID": "UnitID",
    "Incident Number": "IncidentNumber",
    "Call Type": "CallType",
    "Call Date": "CallDate",
    "Watch Date": "WatchDate",
    "Received DtTm": "ReceivedDtTm",
    "Entry DtTm": "EntryDtTm",
    "Dispatch DtTm": "DispatchDtTm",
    "Response DtTm": "ResponseDtTm",
    "On Scene DtTm": "OnSceneDtTm",
    "Transport DtTm": "TransportDtTm",
    "Hospital DtTm": "HospitalDtTm",
    "Call Final Disposition": "CallFinalDisposition",
    "Available DtTm": "AvailableDtTm",
    "Zipcode of Incident": "ZipcodeofIncident",
    "Station Area": "StationArea",
    "Original Priority": "OriginalPriority",
    "FinalPriority": "FinalPriority",
    "ALS Unit": "ALSUnit",
    "Call Type Group": "CallTypeGroup",
    "Number of Alarms": "NumberofAlarms",
    "Unit Type": "UnitType",
    "Unit sequence in call dispatch": "Unitsequenceincalldispatch",
    "Fire Prevention District": "FirePreventionDistrict",
    "Supervisor District": "SupervisorDistrict",
    "Neighborhooods - Analysis Boundaries": "neighborhoods_analysis_boundaries"
}

for old, new in rename_map.items():
    fire_df = fire_df.withColumnRenamed(old, new)
```

---

## 🧪 3. 建立暫存表供 SQL 查詢

```python
fire_df.createOrReplaceTempView("fire_calls")
```

你可以在 Spark SQL 中查詢：

```sql
spark.sql(SELECT * FROM fire_calls LIMIT 5).show()
```

---

## 🧾 4. 資料結構與預覽

```python
fire_df.printSchema()
fire_df.show(2, truncate=False, vertical=True)
```

---

## 📊 5. 分析：2018 年每週通報事件數（週次統計）

```python
from pyspark.sql.functions import to_date, year, weekofyear, col

result = spark.table("fire_calls") \
    .withColumn("call_date", to_date(col("CallDate"), "MM/dd/yyyy")) \
    .filter(year(col("call_date")) == 2018) \
    .withColumn("week_of_year", weekofyear(col("call_date"))) \
    .groupBy("week_of_year") \
    .count() \
    .select("week_of_year", "count") \
    .orderBy(col("count").desc())

result.show()
```

### 📘 筆記：

- `to_date(...)`：轉換字串日期格式
- `year(...)`：擷取年份
- `weekofyear(...)`：擷取週次（1~52）
- `groupBy(...).count()`：統計每週通報數
- `orderBy(...desc())`：依數量排序

---

## ✅ 延伸建議

- 若要輸出為 CSV：
  ```python
  result.write.csv("output/fire_calls_by_week.csv", header=True)
  ```

- 若要畫出趨勢圖，可用：
  ```python
  result.toPandas().plot(x="week_of_year", y="count", kind="bar")
  ```

---

## Reference
[PySpark - Apache Spark Programming in Python for beginners](https://www.udemy.com/course/apache-spark-programming-in-python-for-beginners/)
