_Notes on Spark Streaming Lesson 2 | September 2020_ 

# Data Wrangling with Spark

---

## Functional Programming

Spark uses functional programming rather than procedural programming (Python).

Functional programming is perfect for distributed system. It helps minimize mistakes due to its well-defined input output requirements. There could also be side effects in procedural methods. 

Spark uses lazy evalution. Before it performs calculation, it will optimize the steps that it will take, procastinating the calculation step until required. In Spark, the steps are called stages. 

## SparkSession

For lower level abstraction, such as when handling RDD, use Spark Context

```
configure = SparkConf().setAppName("name").setmaster("IP Address")

sc = SparkContext(conf = configure)
```

Otherwise, use Spark Session:
```
spark = SparkSession \
.builder \
.appName("app name") \
.config("config option", "config value") \
.getOrCreate()
```

### Spark functions

- `select()`: returns a new DataFrame with the selected columns
- `filter()`: filters rows using the given condition
- `where()`: is just an alias for filter()
- `groupBy()`: groups the DataFrame using the specified columns, so we can run aggregation on them
- `sort()`: returns a new DataFrame sorted by the specified column(s). By default the second parameter 'ascending' is True.
- `dropDuplicates()`: returns a new DataFrame with unique rows based on all or just a subset of columns
- `withColumn()`: returns a new DataFrame by adding a column or replacing the existing column that has the same name. The first parameter is the name of the new column, the second is an expression of how to compute it.

Spark SQL provides built-in methods for the most common aggregations such as `count()`, `countDistinct()`, `avg()`, `max()`, `min()`, etc. in the pyspark.sql.functions module. These methods are not the same as the built-in methods in the Python Standard Library, where we can find `min()` for example as well, hence you need to be careful not to use them interchangeably.

In many cases, there are multiple ways to express the same aggregations. For example, if we would like to compute one type of aggregate for one or more columns of the DataFrame we can just simply chain the aggregate method after a `groupBy()`. If we would like to use different functions on different columns, `agg()` comes in handy. For example `agg({"salary": "avg", "age": "max"})` computes the average salary and maximum age.

We can also define our own functions in Spark using the udf method from pyspark.sql.functions module. 

Spark also supports window functions with similar use method as in SQL. When defining the window we can choose how to sort and group (with the partitionBy method) the rows and how wide of a window we'd like to use (described by rangeBetween or rowsBetween).

### Spark SQL

Example usage: First build the temporary view/table, then query the table

`user_df.createOrReplaceTempView("user_log_table")`
`spark.sql("SELECT * FROM user_log_table LIMIT 2").show()`
`spark.sql("SELECT *, SUM(is_home) OVER \
    (PARTITION BY userID ORDER BY ts DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS period \
    FROM is_home_table")`
`spark.sql("SELECT userID, page, ts, CASE WHEN page = 'Home' THEN 1 ELSE 0 END AS is_home FROM log_table \
            WHERE (page = 'NextSong') or (page = 'Home') \
            ")`

### RDD

RDDs are a low-level abstraction of the data. RDDs can be thought as long lists distributed across various machines. You can still use RDDs as part of your Spark code although data frames and SQL are easier. 

### Resources

- [Spark SQL guide](https://spark.apache.org/docs/latest/sql-getting-started.html)
- [RDD vs DataFrames vs Datasets](https://databricks.com/blog/2016/07/14/a-tale-of-three-apache-spark-apis-rdds-dataframes-and-datasets.html)