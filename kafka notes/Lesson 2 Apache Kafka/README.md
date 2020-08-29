_Notes on Kafka Lesson 2 | August 2020_ 

# Glossary

- **Broker (Kafka)** - A single member server of the Kafka cluster
- **Cluster (Kafka)** - A group of one or more Kafka Brokers working together to satisfy Kafka production and consumption
- **Node** - A single computing instance. May be physical, as in a server in a datacenter, or virtual, as an instance might be in AWS, GCP, or Azure.
- **Zookeeper** - Used by Kafka Brokers to determine which broker is the leader of a given partition and topic, as well as track cluster membership and configuration for Kafka
- **Access Control List (ACL)** - Permissions associated with an object. In Kafka, this typically refers to a user’s permissions with respect to production and consumption, and/or the topics themselves.
- **JVM** - The Java Virtual Machine - Responsible for allowing host computers to execute the byte-code compiled against the JVM.
- **Data Partition (Kafka)** - Kafka topics consist of one or more partitions. A partition is a log which provides ordering guarantees for all of the data contained within it. Partitions are chosen by hashing key values.
- **Data Replication (Kafka)** - A mechanism by which data is written to more than one broker to ensure that if a single broker is lost, a replicated copy of the data is available.
- **In-Sync Replica (ISR)** - A broker which is up to date with the leader for a particular broker for all of the messages in the current topic. This number may be less than the replication factor for a topic.
- **Rebalance** - A process in which the current set of consumers changes (addition or removal of consumer). When this occurs, assignment of partitions to the various consumers in a consumer group must be changed.
- **Data Expiration** - A process in which data is removed from a Topic log, determined by data retention policies.
- **Data Retention** - Policies that determine how long data should be kept. Configured by time or size.
- **Batch Size** - The number of messages that are sent or received from Kafka
- `acks` - The number of broker acknowledgements that must be received from Kafka before a producer continues processing
- **Synchronous Production** - Producers which send a message and wait for a response before performing additional processing
- **Asynchronous Production** - Producers which send a message and do not wait for a response before performing additional processing
- **Avro** - A binary message serialization format
- **Message Serialization** - The process of transforming an application's internal data representation to a format suitable for interprocess communication over a protocol like TCP or HTTP.
- **Message Deserialization** - The process of transforming an incoming set of data from a form suitable for interprocess communication, into a data representation more suitable for the application receiving the data.
- **Retries (Kafka Producer)** - The number of times the underlying library will attempt to deliver data before moving on
- **Consumer Offset** - A value indicating the last seen and processed message of a given consumer, by ID.
- **Consumer Group** - A collection of one or more consumers, identified by group.id which collaborate to consume data from Kafka and share a consumer offset.
- **Consumer Group Coordinator** - The broker in charge of working with the Consumer Group Leader to initiate a rebalance
- **Consumer Group Leader** - The consumer in charge of working with the Group Coordinator to manage the consumer group
- **Topic Subscription** - Kafka consumers indicate to the Kafka Cluster that they would like to consume from one or more topics by specifying one or more topics that they wish to subscribe to
- **Consumer Lag** - The difference between the offset of a consumer group and the latest message offset in Kafka itself
- **CCPA** - California Consumer Privacy Act
- **GDPR** - General Data Protection Regulation

---

# Apache Kafka

Kafka is built on **JVM**.

Kafka servers are referred to as **brokers**. Clusters may consist of one or many brokers.

Apache Zookeeper is used by Kafka to determine which broker is the leader of a given partition and topic. 

Zookeeper keeps track of which brokers are part of Kafka cluster. Zookeeper stores configuration for topics and permissions.

Zookeeper allows broker to enter and leave a cluster easily (grow easily as usage increases).


<p align="center"><img src="../images/zookeeper.png" height= "300"/></p>


Kafka stores data in text files in broker disk.

<p align="center"><img src="../images/file_storage.png" height= "300"/></p>

### Kafka partitions

Kafka is partitioned to enable high throughput (parallel processing).

A partition contains an ordered subset of data in the topic.

Every partition has a single leader broker, elected by Zookeeper. Producers evenly hash data to partitions.

<p align="center"><img src="../images/partitions_broker.png" height= "300"/></p>

### Kafka replication

Data Replication helps prevent data loss by writing the same data to more than one broker.

When a leader broker for a partition is lost, a new leader is elected by Zookeeper. 

Number of replica can be configured (globally or for each topic).

Can't have more replicas than number of brokers.

### Checking data in Kafka

Data in Kafka is stored in the `var/lib/lab/kafka/data/` directory.

### Kafka Topics

A broker must be an In Sync Replica (ISR) to become leader.

Desired number of ISRs can be set on topics. Be careful not to set it too high (slow down processing). 

Ordering is guaranteed only within a topic's partitions.

Determine the number of partitions: 
> Number of Partitions = Max(Overall Throughput/Producer Throughput, Overall Throughput/Consumer Throughput)

Kafka naming convention: 
- alphanumeric characters (a-z, A-Z, 0-9, ".", "-")
- Example `com.udacity.lesson.quiz_complete`
- where `com.udacity` is the domain, `lesson` is the model, and `quiz_complete` is the event

Topics may expire based on time and size limit. Using `retention.bytes` and `retention.ms` settings. When data expires it is deleted (set using `cleanup.policy` set to `delete`)

Log compaction removes repeat messages in Kafka topic. (`cleanup.policy` set to `compact`)

Compression options are available using lz4, ztsd, snappy, gzip (using `compression.type` setting)

**Never put more than one event in a single topic**. Topic should store data for one type of event only.

### Creating Topics in Kafka

Always manually create topic (or use bash). Don't rely on automatic topic creation. 

## Kafka Producers

Synchronous producers (rare): send data to Kafka and block program until broker has confirmed receipt
- waiting for delivery of credit card transaction before moving forward
- not the default
- used where data loss cannot be tolerated

Asynchronous producers (common): send data and immediately continue
- maximize throughput to Kafka
- can offer callbacks when message is delivered or if error occurs
- can also fire and forget and don't check for broker receiving data

Use `Producer().flush()` to wait for message to be delivered (synchronous)

#### Message Serialization

Transforming data from internal representation into a format suitable for data store (e.g. Avro, CSV).
- most common serialization type binary, csv, json, avro
- never mix serialization type in a topic (create new one if you wish to change) 

#### Producer Configuration

[Configuration options for kafka python](https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md)

Always set `client.id` for improved logging and debugging!

Configure `retries` to ensure data is delivered.

Set `enable.idempotence` to `True` (idempotence = process which does not change result if happens twice)

If `compression.type` is set on the producer it is done on the client machine, if compression is set on broker, it is done on broker.
- If both are set then two compressions are performed (wasteful)

`acks` determine number of required In Sync Replica (ISR) confirmations. (number of replica to receive message from client before before moving on). 
- set default to `all` or `-1`
- don't add this complexity unless for high throughput and performance begin to lag



#### Resources

- [What is Zookeeper](https://www.cloudkarafka.com/blog/2018-07-04-cloudkarafka_what_is_zookeeper.html)
- [Kafka Design motivation](https://kafka.apache.org/documentation/#design)
