_Notes on Spark Streaming Lesson 4 | September 2020_ 

# Structured Streaming APIs

# Glossary

- **Lazy evaluation**: An evaluation method for expressions in which expressions are not evaluated until their value is needed.
- **Action**: A type of function for RDDs/DataFrames/Datasets where the **values are sent to the driver. A new RDD/DataFrame/Dataset is not formed**.
- **Transformation**: A lazily evaluated function where **a new RDD/DataFrame/Dataset is formed***. There are narrow and wide types of transformations - narrow transformations occur within the same partition, whereas wide transformations may occur across all partitions.
- **Sink**: Place for streaming writes (output).
---

## RDD/DataFrame Functions 

There are two types of Apache Spark RDD operations - transformations and actions. 

Transformations produce a new RDD from an existing RDD. Actions trigger evaluation of expressions and send data back to the driver.

Transformations build an RDD lineage. For example, this code `rdd.map().filter().join().repartition()` builds a lineage of map -> filter -> join -> repartition. You can see this lineage through the .explain operation or through the Spark UI.

The resulting RDD from a transformation is always different from its parent RDD. It can be smaller or bigger or the same size (e.g. map).

Transformations also have two types of operations - **narrow and wide**. Narrow is map-like, and wide is reduce-like.

You may persist an RDD in memory using the **persist** method if there are many transformations and actions applied, so that Spark keeps the RDD around the cluster for faster access the next time.

### Lazy Evaluation

Lazy evaluation means an expression is not evaluated until a certain condition is met. In Spark, this is when an action triggers the DAG. Transformations are lazy and do not execute immediately. 

Spark adds them to a DAG of computation, and only when the driver requests some data (with an action function) does this DAG actually get executed.

Advantages of lazy evaluation:
- Users can organize their Apache Spark program into smaller operations. It reduces the number of passes on data by grouping operations.
- Saves resources by not executing every step. It saves the network trips between driver and cluster. This also saves time.

### Transformations

- Transformations are expressions that are lazily evaluated and can be chained together. Then, as we discussed in the previous lesson, Spark Catalyst will use these chains to define stages, based on its optimization algorithm.

- There are two types of transformations - wide and narrow. Narrow transformations have the parent RDDs in the same partition, whereas parent RDDs of the wide transformations may not be in the same partition.

### Actions

Actions are operations that produce non-RDD values. Actions send the values from the executors to be stored to the driver.

Unlike transformations, which are lazily evaluated, actions evaluate the expression on RDD.

A few action functions that are used often are:

- save()
- collect() - return entire content of RDD into driver program
- take(n) - return n number of elements, it will return different results each time
- foreach() - to apply operation to each element of RDD but not return value to driver
- count()
- reduce()
- getNumPartitions()
- aggregate()

<p align="center"><img src="images/common_transformations.png" height= "185"/></p>

### Structured Streaming APIs

<p align="center"><img src="images/join_types.png" height= "185"/></p>

Full outer joins not supported for streams as events may arrive later than the original timestamps and thus may create unwanted effects in the final table. 

Watermark handles two problems: 
- determine which data can be accepted in the popeline
- discard state that is too old from the state store

Specify two columns to compute watermark:
- column defining event time
- value for delay threshold

Watermark allows late data to be reaggregated into past groups. We need to specify the delay-threshold or the amount of time the computer will wait for old data. 

Watermark decides which data can be dropped or included. We can achieve stateful aggregation using watermark.

Example code:
```
    # TODO use watermark of 10 seconds
    left_with_watermark = left \
        .selectExpr("row_id AS left_row_id", "left_timestamp") \
        .withWatermark("left_timestamp", "10 seconds")
```

<p align="center"><img src="images/watermark.png" height= "300"/></p>

### Query plans

We can apply `.explain()` at the end of a query to visualize the query plan. 

- FileScan informs how the original file was scanned and loaded into the RDD. You can see the format of the file, the location of the file, and the column names (if appropriate).
- Spark uses BroadcastHashJoin when the size of the data is below BroadcastJoinThreshold. In the demo, we’re using small sized data and thus we see that the JOIN query is using BroadcastHashJoin.

### Writing to ouput sinks

There are a few modes for writing to output sinks:

- Append - Only new rows are written to the output sink.
- Complete - All the rows are written to the output sink. This mode is used with aggregations.
- Update - Similar to complete, but only the updated rows will be written to the output sink.

The usefulness of an output sink is to support pipeline fault tolerance by saving output to a stable destination.

### State management

State management became important when we entered the streaming realm. You always want to save the metadata and data for the state for many purposes - like logging, metrics, metadata about your data, etc.

Stateless transformations are like map(), filter(), and reduceByKey(). Each DStream is a continuous stream of RDDs. This type of transformation is applied to each RDD.

Stateful transformations track data across time. This means that the stateful transformation requires some shuffles between keys in key/value pair. The two main types of shuffles are windowed operations:
- `updateStateByKey()` - Used to track state across events for each key. updateStateByKey() iterates over all incoming batches and affects performance when dealing with a large dataset.
- `mapWithState()` - Only considers a single batch at a time and provides timeout mechanism.

Checkpointing versus Persisting:
- Persist keeps the RDD lineage
- Checkpoints does not store the RDD lineage
- Checkpoint requires a location in the computer to write to disk

### Stateful vs Stateless

State means the intermediate data maintained between records.

Stateful stream processing means past events can affect the way current and future events are processed. For example, aggregations (total count of a key in a given interval)

Stateful systems can keep track of windows and perform counts. These can be checkpointed.  Checkpoint location has to be a distributed system, like HDFS, not in C drive.

Stateless stream processing is easy to scale up because events are processed independently. Can launch multiple processors to deal with additional throughput.

We always want to store state when engaged in stateful stream processing.

We always want to use checkpointing when there are changes in state. (e.g. aggregations over a moving interval)

**Importance of state store**

The purpose of state store is to provide a reliable place in your services so that the application (or the developer) can read the intermediary result of stateful aggregations.

In the case of driver or worker failures, Spark is able to recover the processing state at the point right before the failure. 

Can use in-memory hashmap, HDFS, Cassandra, AWS, DynamoDB. State store has been implemented in Spark Structured Streaming. Users can recover from the point of failure much more easily now. 

The state stored is supported by HDFS compatible file system. To guarantee recoverability, Spark recovers the two most recent versions.

State store by implementing `org.apache.spark.sql.execution.streaming.state.StateStore properties`

Example code for checkpointing:
```
from pyspark.sql import SparkSession

def checkpoint_exercise():
    """
    note that this code will not run in the classroom workspace because we don't have HDFS-compatible file system
    :return:
    """
    spark = SparkSession.builder \
            .master("local") \
            .appName("Checkpoint Example") \
            .getOrCreate()

    df = spark.readStream \
        .format("rate") \
        .option("rowsPerSecond", 90000) \
        .option("rampUpTime", 1) \
        .load()

    rate_raw_data = df.selectExpr("CAST(timestamp AS STRING)", "CAST(value AS string)")

    stream_query = rate_raw_data.writeStream \
        .format("console") \
        .queryName("Default") \
        .option("checkpointLocation", "/tmp/checkpoint") \ #this checkpoint location requires HDFS-like filesystem
        .start()

if __name__ == "__main__":
    checkpoint_exercise()

```

### Troubleshooting OutofMemory Error

Can happen when data to process is too large or too many shuffles is occurring. 

We can use properties like `Spark.executor.memory` and `Spark.driver.memory` to set the amount of memory to allocate.

We can set `spark.sql.shuffle.partitions` configures the number of partitions when shuffling such as aggregation. `spark.default.parallelism` returns the default number of RDD partitions from operations such as map or flatmap.

If the job takes too long, we can increase the number of nodes.

### Key considerations

There are a few ways to tune your Spark Structured Streaming application. But before that, go through your application and try to answer these questions.

- Study the memory available for your application. Do you have enough memory to process your Spark job? If not, consider vertical scaling (improve hardware). If you do have enough memory but limited resources, consider horizontal scaling(add more machines).
- Study your query plans - do they make sense? Are you doing unnecessary shuffles/aggregations? Can you reduce your shuffles?
- What’s the throughput/latency of your data?
- How are you saving your data? Are you persisting your data to memory or to disk only, or to both memory and disk?
- Are you breaking your lineage anywhere?

### Resources

- [Stateful Stream Processing](https://databricks.com/blog/2016/02/01/faster-stateful-stream-processing-in-apache-spark-streaming.html)
- [Stream-stream joins](https://databricks.com/blog/2018/03/13/introducing-stream-stream-joins-in-apache-spark-2-3.html)
- [Spark journal](https://cacm.acm.org/magazines/2016/11/209116-apache-spark/fulltext)