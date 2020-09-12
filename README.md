# Udacity Data Streaming Nanodegree

Projects completed in the Udacity [Data Streaming Nanodegree](https://www.udacity.com/course/data-streaming-nanodegree--nd029) program.

## Project 1: [Optimizing Public Transportation](https://github.com/ggbong734/udacity-data-streaming/tree/master/Project%20-%20Optimizing%20Public%20Transportation)

<p align="center"><img src="Project - Optimizing Public Transportation/images/apache_kafka.png" height= "185"/></p>

<p align="center"><img src="Project - Optimizing Public Transportation/images/faust_logo.jpg" height= "140"/></p>

Constructed a streaming event pipeline around Apache Kafka and its ecosystem.
- Configured the producers such that events are sent to Kafka along with the Avro key and value schemas
- Ingested data from a PostgreSQL database using Kafka Connect
- Utilized Faust to transform ingested data 
- Aggregated data using KSQL 
- Configured the consumer to consume data from Kafka

Proficiencies used: Apache Kafka, Kafka Connect, Faust Stream processing, KSQL

## Project 2: [SF Crime Statistics with Spark Streaming](https://github.com/ggbong734/udacity-data-streaming/tree/master/Project%20-%20SF%20Crime%20Statistics)

<p align="center"><img src="Notes - Spark Streaming\images\spark_streaming.png" height= "185"/></p>

Created a Kafka server to produce data and ingested the data using Spark Structured Streaming.
- Built a simple Kafka server (with Zookeeper) for producing data
- Created a Spark consumer to perform data aggregation and joining
- Modified SparkSession property parameters to optimize processing throughput

Proficiencies used: Apache Kafka, Apache Spark, Spark Structured Streaming