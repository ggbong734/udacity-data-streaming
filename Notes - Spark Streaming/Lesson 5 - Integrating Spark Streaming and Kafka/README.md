_Notes on Spark Streaming Lesson 5 | September 2020_ 

# Integration of Spark Streaming and Kafka

# Glossary

- **Write Ahead Logs (WAL)**: This is where the operation is logged in a file format. When unfortunate events like driver or node failure happen and the application is restarted, the operation logged in the WAL can be applied to the data.
- **Broadcasting**: Spark allows users to keep a read-only variable cached on each machine rather than sending the data over the network with tasks. This is useful when you have a rather large dataset and don't want to send the dataset over the network.
- **Salting**: Another popular method of redistributing a dataset. This is done by adding a "fake-primary-key" to redistribute the data.
---

## Kafka Producer Server

Using KafkaProducer to create a producer and push json data from a text file to a Kafka topic.
``` 
from kafka import KafkaProducer
import json
import time

class ProducerServer(KafkaProducer):

    def __init__(self, input_file, topic, **kwargs):
        super().__init__(**kwargs)
        self.input_file = input_file
        self.topic = topic
        
    def generate_data(self):
        with open(self.input_file) as f:
            for line in f:
                message = self.dict_to_binary(line)
                self.send(self.topic, message)
                time.sleep(1)
                
    def dict_to_binary(self, json_dict):
        return json.dumps(json_dict).encode('utf-8')
```

### Kafka Source Provider

KafkaSourceProvider provides a consumer for Kafka within Spark, therefore we will not need to create separate consumer modules for consumption.

KafkaSourceProvider requires these options
- subscribe, subscribepattern, or assign
- kafka.bootstrap.server

```
kafka_df = spark.readStream.\
  format("kafka").\ # set data ingestion format as Kafka
  option("subscribe", "<topic_name>").\ #This is required although it says option.
  option("kafka.bootstrap.servers", "localhost:9092").\ #You will also need the url and port of the bootstrap server
  load()
```

### Kafka Offset Reader 

Managing offsets becomes crucial to making your Kafka and Spark microservices be best optimized.

KafkaOffsetReader is a class within Spark that uses Kafka's own KafkaConsumer API to read Kafka offsets.

It helps define which Kafka topic and partitions should be read.

Kafka offsets can be stored to take care of lag issues to keep track of which messages have been read already. 

Offset can be stored at HDFS, S3, ...

### Triggers in Spark Streaming

Write Ahead Logs enforce fault-tolerance by saving logs to a certain checkpoint directory. Enabling WAL can be done by using the Spark property, `spark.streaming.receiver.writeAheadLog.enable`.

Spark Triggers determine how often a streaming query needs to be executed. The trigger can be set using a few options in the query builder.

Default behavior in Spark Streaming is to read streaming data in micro batches.

Example of incorporating trigger in stream:
```
    query = agg_df \
        .writeStream \
        .trigger(processingTime="5 seconds") \
        .outputMode('Complete') \
        .format('console') \
        .option("truncate", "false") \
        .start()
```

Example of how Kafka can be integrated with Spark:

<p align="center"><img src="images/kafka_with_spark.png" height= "185"/></p>

 Kafka provides data streams of events and a Spark application will often be integrated to process the stream of data. The sinks should be implemented wherever you want to save your intermediary data, for example, at the end of each micro-batch.

### Structured Streaming on Spark UI

The “Streaming” tab in Spark UI shows what kind of streaming data you’re ingesting from (in the example below, Kafka stream), number of offsets in each partition, and also input rate of the data.

### Spark Progress Reports

<p align="center"><img src="images/spark_progress_report.png" height= "210"/></p>

What we really need to take a look at are these three pieces of data, because they provide important information regarding the batch size and processed rows:

- `numInputRows` : The aggregate (across all sources) number of records processed in a trigger.
- `inputRowsPerSecond` : The aggregate (across all sources) rate of data arriving.
- `processedRowsPerSecond` : The aggregate (across all sources) rate at which Spark is processing data.

Additionally `Sink`: Streaming sink of the streaming query

### Kafka-Spark Integration Scenario 1

**Scenario 1**
Given problem: We're given a hypothetical Spark streaming application. This application receives data from Kafka. Every 2 minutes, you can see that Kafka is producing 60000 records. But at its maximum load, the application takes between 2 minutes for each micro-batch of 500 records. How do we improve the speed of this application?

We can tweak the application's algorithm to speed up the application.

Let's say the application's algorithm is tweaked - how can you check if most or all of the CPU cores are working?
In a Spark Streaming job, **Kafka partitions map 1:1 with Spark partitions. So we can increase Parallelism by increasing the number of partitions in Kafka, which then will increase Spark partitions**.

We can **check if the input data was balanced/unbalanced, skewed or not.** We can **check the throughput of each partition using Spark UI, and how many cores are consistently working**. You can also use the `htop` command to see if your cores are all working (if you have a small cluster).

**Increase driver and executor memory: Out-of-memory issues can be frequently solved by increasing the memory of executor and driver**. Always try to give some overhead (usually 10%) to fit your excessive applications.
You could also **set `spark.streaming.kafka.maxRatePerPartition` to a higher number** and see if there is any increase in data ingestion.

**Scenario 2**

Other methods of improving your Spark application are by improving system stability and studying your data.

When the heap gets very big (> 32GB), the cost of GC (garbage collection) gets large as well. We might see this when the join application runs with large shuffles (> 20GB), and the GC time will spike.

In the real world, uneven partitioning is unavoidable due to the nature of your dataset.

Using the Spark UI (or even just your logs), you'll commonly see the errors below:

- Frozen stages
- Low utilization of CPU (workers not working)
- Out of memory errors

To minimize the skewed data problem, we can try the following:

- **Broadcasting (during joining)**: You can increase the autoBroadcastJoinThreshold value in spark.sql property so that **the smaller tables get “broadcasted.”** This is helpful when you have a large dataset that will not fit into your memory.

- **Salting**: If you have a key with high cardinality, your dataset will be skewed. Now you introduce a “salt,” modifying original keys in a certain way and using hash partitioning for proper redistribution of records.

Now that we have cleaned up our data and the skew problem as much as we could, and also assuming that our code is optimized, let’s talk about how we can stabilize the system through a couple of different methods:

- Auto scaling
- Speculative execution

Auto scaling is only doable with cloud clusters as you **can always add more nodes freely**. Two popular tech stacks that are used are AWS Auto Scaling (if AWS EMR clusters are used) or auto scaling with Kubernetes (a container-orchestration system).

Speculative execution is another popular addition to stabilize and reduce bottleneck-like threshold. Speculative execution in Spark detects the “speculated task” (which means this task is running slower than the median speed of the other tasks), and then **submits the “speculated task” to another worker**. This enables continuous parallel execution rather than shutting off the slow task.

### Resources

- [Tuning Spark](https://spark.apache.org/docs/latest/tuning.html)
- [Java Memory Management on JVM](https://dzone.com/articles/java-memory-management)