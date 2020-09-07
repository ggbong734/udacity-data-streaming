_This is project 1 of Udacity's Data Streaming Nanodegree program | Completed in September 2020_

# Data Streaming Project - Apache Kafka Ecosystem

The goal of the project is to construct a streaming event pipeline around Apache Kafka and its ecosystem.

The (simulated) streamed data will be used to continuously update a dashboard which displays the status of train lines in real time. 

![Final product](images/final_product.png)

Technology used in this project:
- Apache Kafka
- Kafka Connect
- Faust Stream Processors
- KSQL

> Note that the project comes with starter code and files. The main tasks in the project were to fill in key components in the `producers` and `consumers` files.

## Project architecture

The architecture of the project is as follows:

![Final product](images/architecture.png)

1. Station information is extracted from a PostgreSQL database into Kafka using Kafka JDBC Source Connector.
2. The raw station data ingested from Kafka Connect is transformed using Faust. 
3. The `producers` code is configured such that arrival events are sent to Kafka. Events emitted to Kafka are paired with the Avro key and value schemas. A topic is created for each train line and the turnstiles. 
4. Weather information is pulled from the weather hardware using HTTP REST and Kafka's REST Proxy. 
5. Turnstile data for each station is aggregated using KSQL to generate the total count per station.
6. The `consumers`, in this case the web server serving the transit status dashboard, consumes the data from Kafka. 

## Running the simulation

There are two main pieces to the code, the `producers` and `consumer`.

To start producing data, run `python simulation.py` in the producers directory.

To start the Faust Stream Processing application, run  `faust -A faust_stream worker -l info` in the consumers directory.

To run the KSQL creation script, run `python ksql.py` in the consumers directory.

To run the consumer, run `python server.py` in the consumers directory. 

## Suggested improvements

- [Kafka topics naming conventions `<message type>.<dataset name>.<data name>`](https://riccomini.name/how-paint-bike-shed-kafka-topic-naming-conventions)

- [Kafka best practices](https://blog.newrelic.com/engineering/kafka-best-practices/)

- [Schema Registry documentation](https://docs.confluent.io/current/schema-registry/index.html)

- [Kafka Python Client use cases and examples](https://github.com/confluentinc/confluent-kafka-python#usage)