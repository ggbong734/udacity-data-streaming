_Notes on Kafka | August 2020_ 

# Data Streaming via Kafka

### Intro

Stream Processing acts on potentially endless and constantly evolving **immutable data** contained in data streams.

Once data have been placed in a data stream, **they cannot be modified**. We must place a new record in the stream to override the existing data.

Data sent to data streams is typically small, less than 1MB in size. **Data volume varies** from a few records an hour to thousands of requests per second.

**Event** - An immutable fact regarding something that has occurred in our system. (For e.g. click event) cannot be changed. 

Example application of stream processing:
- log analysis
- web analytics (clicks and page views)
- real time pricing (affected by environmental factors and instantaneous demand)
- financial analysis (stocks)

Batch and Stream processing are **not mutually exclusive**. Batch systems can create events to feed into stream processing applications, and vice versa.

#### Batch Processing 

- Runs on a scheduled basis
- May run for a longer period of time and write results to a SQL-like store
- May analyze all historical data at once
- Typically works with mutable data and data stores

#### Stream Processing

Stream processing consist of a stream data store (e.g. Kafka) and a stream processing application framework (e.g. Kafka Streams)

- Runs at whatever frequency events are generated
- Typically runs quickly, updating in-memory aggregates
- Stream Processing applications may simply emit events themselves, rather than write to an event store
- Typically analyzes trends over a limited period of time due to data volume
- Typically analyzes immutable data and data stores

**Benefits**
- faster
- more scalabe (distributed nature of data stores)
- decouples how data is used to how it is produced
- immutability provides easier fault tolerance

Streaming data stores guarantee that **data is stored in the order it was produced**. Also guarantees that **events stored are unchageable**. E.g. Kafka (like message queue), Cassandra (like SQL store)

**Change Data Capture process** is used in SQL databases using append-only logs to keep nodes and its replicas in sync. Insert, update, delete events are stored in the log. If a failure occurs, the append-only log (or write ahead log in PostGres) is used to bring the failed node up to speed.

**Log-structured storage** 
- consists of append-only logs on disk
- E.g. Cassandra, HBase, Kafka

### Apache Kafka
- message queue interface on top of its append-only log-structured storage medium
- Kafka is a log of events
- an event is something that has happened, as opposed to a request
- data is distributed to multiple nodes by default
- Fault tolerant if a node is lost and provide ordering guarantees for data stored
- now maintained by Confluent

**Kafka Topic**
- used to organize and segment datasets (similar to SQL tables)
- Kafka Topics are not queryable
- data in key-value ninary format
- must be created from CLI (command line interface) or automatically

**Kafka Producer**
- Applications that send data into Kafka Topic
- Producers often built with a client library

**Kafka Consumer**
- Applications that pull data from one or more Kafka Topics
- Integrate with Kafka via a Client Library written in Python/Java/Go
- Only consume data produced after connecting to topic (not all historical data)

### Kafka on Command Line Interface

`kafka-topics --list --zookeeper localhost:2181` to **display all topics**.`--zookeeper` is needed so that the CLI tool can talk to Kafka cluster.

`kafka-topics --create --topic "my-first-topic" --partitions 1 --replication-factor 1 --zookeeper localhost:2181` to **create a new topic**. 

`kafka-console-producer --topic "my-first-topic" --broker-list PLAINTEXT://localhost:9092` to **produce data to the topic**.  `--broker-list` is needed to tell Kafka which broker to connect to.

`kafka-console-consumer --topic "my-first-topic" --bootstrap-server PLAINTEXT://localhost:9092 --from-beginning` to**consume data from a topic**. Note that `--bootstrap-server` instead of `--broker-list` is used to connect to the broker. Add the switch `--from-beginning` to show all historical data in the topic.

# Stream Processing in Kafka


# Glossary

- **Stream** - An unbounded sequence of ordered, immutable data
- **Stream Processing** - Continual calculations performed on one or more Streams
- **Immutable Data** - Data that cannot be changed once it has been created
- **Event** - An immutable fact regarding something that has occurred in our system.
- **Batch Processing** - Scheduled, periodic analysis of one or more groups of related data.
- **Data Store** - A generic place that holds data of some kind, like a message queue or data store
- **Stream Processing Application** - An application which is downstream of one or more data streams and performs some kind of - calculation on incoming data, typically producing one or more output data streams
- **Stream Processing Framework** - A set of tools, typically bundled as a library, used to construct a Stream Processing Application
- **Real-time** - In relation to processing, this implies that a piece of data, or an event, is processed almost as soon as it is produced. Strict time-based definitions of real-time are controversial in the industry and vary widely between applications. For example, a Computer Vision application may consider real-time to be 1 millisecond or less, whereas a data engineering team may 
consider it to be 30 seconds or less. In this class when the term "real-time" is used, the time-frame we have in mind is seconds.

- **Append-only Log** - files in which incoming events are written to the end of the file as they are received
- **Change Data Capture (CDC)** - The process of capturing change events, typically in SQL database systems, in order to accurately communicate and synchronize changes from primary to replica nodes in a clustered system.
- **Log-Structured Storage** - Systems built on Append-Only Logs, in which system data is stored in log format.
- **Merge (Log Files)** - When two or more log files are joined together into a single output log file
- **Compact (Log Files)** - When data from one or more files is deleted, typically based on the age of data

- **Source (Kafka)** - A term sometimes used to refer to Kafka clients which are producing data into Kafka, typically in reference to another data store
- **Sink (Kafka)** - A term sometimes used to refer to Kafka clients which are extracting data from Kafka, typically in reference to another data store
- **Topic (Kafka)** - A logical construct used to organize and segment datasets within Kafka, similar to how SQL databases use tables
- **Producer (Kafka)** - An application which is sending data to one or more Kafka Topics.
- **Consumer (Kafka)** - An application which is receiving data from one or more Kafka Topics.

#### Resources

[Uber Engineering Tech Stack Part 1](https://eng.uber.com/tech-stack-part-one-foundation/)
[Uber Engineering Tech Stack Part 2](https://eng.uber.com/uber-tech-stack-part-two/)
