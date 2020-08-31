_Notes on Kafka Lesson 6 | August 2020_ 

# Stream Processing with Faust

# Glossary

- **DSL** - Domain Specific Language. A metaprogramming language for specific tasks, such as building database queries or stream processing applications.
- **Dataclass (Python)** - A special type of Class in which instances are meant to represent data, but not contain mutating functions
- **Changelog** - An append-only log of changes made to a particular component. In the case of Faust and other stream processors, this tracks all changes to a given processor.
- **Processor (Faust)** - Functions that take a value and return a value. Can be added in a pre-defined list of callbacks to stream declarations.
- **Operations (Faust)** - Actions that can be applied to an incoming stream to create an intermediate stream containing some modification, such as a group-by or filter

---

## Faust Stream Processing

Faust was created at Robinhood. Stream processing framework written and usable in Python. 

The goal was to replicate Kafka Streams in Python. Shares conceptual design patterns with Kafka Streams.

Other processing framework in Java and Scala (Kafka Streams). 

The core stream processing tasks are:
- combining
- filtering
- aggregating
- reducing

Faust is a native Python API, not a Domain Specific Language (DSL) for metaprogramming.

Faust requires no external dependency other than Kafka. Does not require resource manager like Yarn or Mesos. 

### Writing Faust application

See Lesson 6 Chapter 5 on how to write Faust application in Python

Snippet:
```
import faust

app = faust.App('hello-world-faust', broker = 'localhost:9092')

topic = app.topic("com.udacity.streams.clickevents")

@app.agent(topic) # each time an event happen on this topic, the event is passed to function
async def clickevent(clickevents):

    #Print each event inside the for loop
    async for clickevent in clickevents:
        print(clickevent)

if __name__ == "__main__":
    app.main()

```

Run `python filename.py worker` in terminal to start printing the clickevents.

Every Faust application as an app which instantiates the faust application. 

The application must be assigned a topic to subscribe to.

An output table or stream received the output of the processing. 

Asynchronous function is decorated with an `agent`

### Serialization and Deserialization

Dataclasses are new in Python 3.7. 

Dataclass is a special type of Class instance. They can be marked as frozen, which makes them immutable. 

```
from dataclasses import dataclass

@dataclass(frozen=True)
class Purchase:
	username: str = "default"
	currency: str = ""
	amount: int = 0
```

dataclass object require type annotations on fields and will enforce these type constraints on creation.

Dataclass can be paired with `asdict` function to quickly transform dataclasses into dictionaries.

Try to use dataclasses when using Faust. It makes clear the structure of the data we want.

### Deserialization

Deserialization is handled by specifying `key_type` and `value_type` to the Faust topic.

E.g. `topic = app.topic("topic_name", key_type = str, value_type = Purchase)`

In order to use data model classes, the Purchase class has to inherit from faust.Record

```
class Purchase (faust.Record, validation = True, serializer = 'json'):
	username: str
	currency: str
```

Unfortunately, at the time of writing, Faust has no support for Avro. Custom deserializer has to be used.

Setting serializer type to json enables Faust to deserialize data in this format by default.

`validation = True` will enforce data being deserialized from Kafka to match the expected type. If we expect `str` but get an `int`, Faust will raise an error. 

### Serialization

Faust Serialization leverages the same `faust.Record`. Faust run the serializer **in reverse** to serialize the data for the output stream. 

Here we have two serializer in succession, json and then base64 encode it.
If we are deserializing we will base64 decode the data and then unpack the json into our model. 

```
class Purchase (faust.Record, validation = True, serializer = 'binary|json'):
	username: str
	currency: str
```

In Faust it is very simple to modify a stream, simply
1. Define the new old and new class with faust.Record
2. Create a new topic for the processed/sanitized data
3. Modify the incoming data using the new object class
4. Send the modified data to the new topic, specifying the key and value

See Lesson 6 Chapter 8 for the example!

### Storage in Faust

Faust stores its changelog in Kafka (in a topic) and uses RocksDB for local state. 

If a fault occurs, Kafka will use the changelog to rebuild state (restore topic to the previous state).

In-memory storage should only be used for test and local development. Data is lost in memory during restarts so Faust has to rebuild the state. If there are a lot of events, it might take a long time to rebuild state which is unacceptable in production. Further, the state may not fit in memory.

The better option to store state locally is to use RocksDB. **RocksDB stores the state on disk**. As changes are made, they are stored in RocksDB and sent to Kafka.  

It is always recommended to use RocksDB in production. Simply need the library installed to make use of it. 

### Message Life Cycle 

Faust streams are simply infinite asynchronous iterables.

Faust handles consumption, consumer groups, and offsets for us, in addition to managing message acknowledgements. 

Faust uses aiokafka to interact with Kafka. 

Faust uses one underlying subscription to topics for all agents. 

Faust applications may forward processed messages to another stream using the `topic.send<data>` function at the end of the processing loop. 

### Filtering streams

Here is an example of how filter can be used to select certain values in stream data. 
```
@app.agent(clickevents_topic)
async def clickevent(clickevents):

    async for ce in clickevents.filter(lambda x: x.number >= 100):

        # Send the message to the `popular_uris_topic` with a key and value.
        await popular_uris_topic.send(key=ce.uri, value=ce)
```

### Stream processors

Processors are functions that take a value and return a value and can be added in a pre-defined list of callbacks to your stream declarations.

Processors promote reusability in your code. 

Processors may execute synchronously or asynchronously using the `asyn` keyword when defining functions.

All defined processors will run in the order they were defined, before the final value is generated.

Example:
```
def add_score(record): 
    record.score = random.random()
    return record

@app.agent(source_topic)
async def record(records):

    records.add_processor(add_score) # add processor callback to stream
    async for re in records:
        await scored_topic.send(key=re.uri, value=re)
```

### Faust Operations

Faust operations are actions that can be applied to an incoming stream to create an intermediate stream containing some modifications such as group by or filter. 

- `group_by` operation ingests every incoming events from a source topic, and emits it to an intermediate topic with the newly specified key
- `filter` operations uses boolean function to determine if a record should be kept.  
- `take` opreation gather up multiple events in the stream before processing them. For example, to take 100 value at a time. But it may hang if the last hundredth message is never received. Need to add `within` `datetime.timedelta` argument to this function. 

## Tables in Faust

Tables have dict-like syntax. 

Tables are defined with `app.table`.

Tables must be co-partitioned with the streams they are aggregating. Use `group_by` to ensure co-partitioning. 

See Lesson 6 Ch. 16 on how to group by. 

### Windowing in Faust

Faust provides two windowing methods: hopping and tumbling. 

```
tumbling_table = table.tumbling(size=timedelta(minutes=5))

hopping_table = table.hopping(
	size=timedelta(minutes=5),
	step=timedelta(minutes=1),
	expires=timedelta(minutes=60))
```

Windowing applies to Tables only.

Faust has semantincs for classifying which pool of data is desired from a window:
- `current()`: get value closest to current local time
- `now()`: get value closest to current local time
- `relative_to_now()`

### Resources

- [Faust](https://faust.readthedocs.io/en/latest/introduction.html)
- [Faust operations](https://faust.readthedocs.io/en/latest/userguide/streams.html#operations)