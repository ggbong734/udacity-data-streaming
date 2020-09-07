_Notes on Spark Streaming Lesson 3 | September 2020_ 

# Intro to Spark Streaming

# Glossary

- **RDD (Resilient Distributed Dataset)** : The fundamental data structure of the Spark Core component. An immutable distributed collection of objects.
- **DataFrame** : A data structure of the Spark SQL component. A distributed collection of data organized into named columns.
- **Dataset** : A data structure of the Spark SQL component. A distributed collection of data organized into named columns and also strongly typed.
- **DAG (Directed Acyclic Graph)**: Each Spark job creates a DAG which consists of task stages to be performed on the clusters.
- **Logical plan** : A pipeline of operations that can be executed as one stage and does not require the data to be shuffled across the partitions — for example, map, filter, etc.
- **Physical plan** : The phase where the action is triggered, and the DAG Scheduler looks at lineage and comes up with the best execution plan with stages and tasks together, and executes the job into a set of tasks in parallel.
- **DAG Scheduler**: DAG scheduler converts a logical execution plan into physical plan.
- **Task** : A task is a unit of work that is sent to the executor.
- **Stage** : A collection of tasks.
- **State** : Intermediary and arbitrary information that needs to be maintained in streaming processing.
- **Lineage Graph**: A complete graph of all the parent RDDs of an RDD. RDD Lineage is built by applying transformations to the RDD.
---

## Spark Components

<p align="center"><img src="images/spark_components.png" height= "185"/></p>

**Core**
- Contains the basic functionality of Spark. Also home to the API that defines RDDs, which is Spark's main programming abstraction.

**SQL**
- Package for working with structured data. It allows querying data via SQL as well as Apache Hive. It supports various sources of data, like Hive tables, Parquet, JSON, CSV, etc.

**Streaming**
- Enables processing of live streams of data. Spark Streaming provides an API for manipulating data streams that are similar to Spark Core's RDD API.

**MLlib**
- Provides multiple types of machine learning algorithms, like classification, regression, clustering, etc. This component will not be a focus of this course.

**GraphX**
- Library for manipulating graphs and performing graph-parallel computations. This library is where you can find PageRank and triangle counting algorithms. This component will not be a focus of this course.

Spark can be run on Hadoop cluster using three methods: standalone, YARN, MESOS

## RDD 

RDD stands for **Resilient Distributed Dataset**:
- **Resilient** because its fault-tolerance comes from maintaining RDD lineage, so even with loss during the operations, you can always go back to where the operation was lost.
- **Distributed** because the data is distributed across many partitions and workers.
- **Dataset** is a collection of partitioned data. RDD has characteristics like in-memory, immutability, lazily evaluated, cacheable, and typed (we don't see this much in Python, but you'd see this in Scala or Java).

Spark RDDs are fault tolerant as they track data lineage information to rebuild lost data automatically on failure.

SparkContext example:
```
from pyspark import SparkConf, SparkContext

conf = SparkConf().setMaster("local[2]").setAppName("RDD Example")
sc = SparkContext(conf=conf)

# different way of setting configurations 
#conf.setMaster('some url')
#conf.set('spark.executor.memory', '2g')
#conf.set('spark.executor.cores', '4')
#conf.set('spark.cores.max', '40')
#conf.set('spark.logConf', True)

# sparkContext.parallelize materializes data into RDD 
# documentation: https://spark.apache.org/docs/2.1.1/programming-guide.html#parallelized-collections
rdd = sc.parallelize([('Richard', 22), ('Alfred', 23), ('Loki',4), ('Albert', 12), ('Alfred', 9)])

rdd.collect() # [('Richard', 22), ('Alfred', 23), ('Loki', 4), ('Albert', 12), ('Alfred', 9)]

# create two different RDDs
left = sc.parallelize([("Richard", 1), ("Alfred", 4)])
right = sc.parallelize([("Richard", 2), ("Alfred", 5)])

joined_rdd = left.join(right)
collected = joined_rdd.collect()

collected #[('Alfred', (4, 5)), ('Richard', (1, 2))]
```

SparkSession example:
```

# Notice we’re using pyspark.sql library here
from pyspark.sql import SparkSession

spark = SparkSession.builder \
        .master("local") \
        .appName("CSV file loader") \
        .getOrCreate()

# couple ways of setting configurations
#spark.conf.set("spark.executor.memory", '8g')
#spark.conf.set('spark.executor.cores', '3')
#spark.conf.set('spark.cores.max', '3')
#spark.conf.set("spark.driver.memory", '8g')

file_path = "./AB_NYC_2019.csv"
# Always load csv files with header=True
df = spark.read.csv(file_path, header=True)

df.printSchema()

df.select('neighbourhood').distinct().show(10, False)
```

## Partitioning in Spark

By default in Spark, a partition is created for each block of the file in HDFS (128MB is the default setting for Hadoop) if you are using HDFS as your file system. If you read a file into an RDD from AWS S3 or some other source, Spark uses 1 partition per 32MB of data. 

There are a few ways to bypass this default upon creation of an RDD, or reshuffling the RDD to resize the number of partitions, by using `rdd.repartition(<the partition number you want to repartition to>)`. For example, `rdd.repartition(10)` should change the number of partitions to 10.

In local mode, Spark uses as many partitions as there are cores, so this will depend on your machine. You can override this by adding a configuration parameter `spark-submit --conf spark.default.parallelism=<some number>`.

#### Hash partitioning

Hash partitioning is defined by

`partition = key.hashCode() % numPartitions`

This mode of partitioning is used when you want to evenly distribute data across partitions.

#### Range partitioning

Range partitioning divides each partition in a continuous but non-overlapping way.

Range partitioning would come into play where you partition the `employees table` by `employee_id`, like this:

```
PARTITION BY RANGE (employee_id) (
    PARTITION p0 VALUES LESS THAN (11),
    PARTITION p0 VALUES LESS THAN (21),
    PARTITION p0 VALUES LESS THAN (31),
    ...
)
```
We would normally use range partition over a timestamp. We can use `partitionByRange()` function to partition data into some kind of groups.

## DataFrames and Datasets 

DataFrames can be thought as tables in a relational database, or dataframes in Python’s pandas library. DataFrames provide memory management and optimized execution plans.

Datasets is strongly typed, unlike DataFrames. They are only available in Java or Scala version of Spark.

RDDs, Datasets, and DataFrames still share common features which are: immutability, resilience, and the capability of distributed computing in-memory.

RDDs are in the Core component, while DataFrames and Datasets are in the SQL component. RDDs and DataFrames are both immutable collections of datasets, but DataFrames are organized into columns. Datasets are organized into columns also, and also provide type safety.

## Intro to Spark Streaming and DStream

Spark DStream, Discretized Stream, is the basic abstraction and building block of Spark Streaming. 

DStream is a continuous stream of RDDs. It receives input from various sources like Kafka, Flume, Kinesis, or TCP sockets. 

<p align="center"><img src="images/dstream.png" height= "150"/></p>

### Spark Structured Streaming

Structured Streaming is built on top of SparkSQL

Internally, Structured Streaming is processed using a micro-batch. It processes data streams as a series of small batch jobs.

Stuctured Streaming application can now guarantee fault-tolerance using checkpointing.

The advantages of using Structured Streaming are:

- Continuous update of the final result
- Can be used in either Scala, Python, or Java
- Computations are optimized due to using the same Spark SQL component (Catalyst)

Structured Streaming now decoupes the state management (saving state to store) and checkpointing metadata. Because these two limitations are decoupled from the application, the developer is now able to exercise fault-tolerant end-to-end-execution with ease, 

### Spark UI/DAGS

Spark UI is a web interface that allows inspection of jobs, stages, storages, environment, and executors in this page, as well as the visualized version of the DAGs (Directed Acyclic Graph) of the Spark job.

Vertices of DAGS are RDDs and the edges are operations like transformation functions applied on RDDs.

DAG is an optimized execution plan with minimized data shuffling. Spark creates a DAG when an action is called and submits to the DAG scheduler. DAG Scheduler divides operators into stages of tasks. Stages are passed to Task Scheduler.

To set port number of Spark UI

`spark = SparkSession.builder.master(local).config("spark.ui.port", 3000)`

The Spark UI becomes very important when you want to debug at system level. It tells you how many stages you’ve created, the amount of resources you’re using, logs, workers, and a lot of other useful information, like lineage graph, duration of tasks, aggregated metrics, etc.

The lineage graph is the history of RDD transformations, and it’s the graph of all the parent RDDs of the current RDD.

Difference between Spark DAGs and Lineage graphs:
> A lineage graph shows you the history of how the final SparkRDD/DataFrame has been created. With the application of transformation functions, you’re building the lineage graph. A DAG shows you different stages of the Spark job. A compilation of DAGs could be the lineage graph, but a DAG contains more information than just stages - it tells you where the shuffles occurred, actions occurred, and with transformations, what kind of RDDs/DataFrames have been generate

### Spark Stages

A stage is a group of tasks. 

### Schema

Example:
```
from pyspark.sql import SparkSession, Row
from pyspark.sql.types import *

schema = StructType([
        StructField("time", TimestampType(), True),
        StructField("address", StringType(), True),
        StructField("phone_number", IntegerType(), True)
    ])

rows = [Row(time= 13211311, address='123 Main Street', phone_number= 1112223333),
            Row(time= 13211313, address='872 Pike Street', phone_number= 8972341253)]

df = spark.createDataFrame(data, schema = schema)
```


### Resources

- [Spark UI](https://databricks.com/blog/2015/06/22/understanding-your-spark-application-through-visualization.html)
- [Project Tungsten](https://databricks.com/blog/2015/04/28/project-tungsten-bringing-spark-closer-to-bare-metal.html)
- [Paper on Whole Stage Codegen](http://www.vldb.org/pvldb/vol4/p539-neumann.pdf)