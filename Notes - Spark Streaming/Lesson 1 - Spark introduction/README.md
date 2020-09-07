_Notes on Spark Streaming Lesson 1 | September 2020_ 

# Spark Introduction

---

## Spark

Spark is currently one of the most popular tools for big data analytics. You might have heard of other tools such as Hadoop. Hadoop is a slightly older technology although still in use by some companies. Spark is generally faster than Hadoop, which is why Spark has become more popular over the last few years.


### Hardware

CPU can store small amounts of data inside itself in what are called **registers**.

When working with big data, the biggest bottlenect is transferring data across a network. Not reading and writing from disk storage (SSD).

Key ratio:
- CPU is 200x faster than memory (RAM)
- Memory is 15x faster than disk (SSD)

Programs typically store data in RAM. By default, if a dataset that we are importing into Python pandas is bigger than the computer's memory, the program won't work. However, the Python pandas library can read in a file in smaller chunks. 

### Hadoop Ecosystem 

**Hadoop** - an ecosystem of tools for big data storage and data analysis. Hadoop is an older system than Spark but is still used by many companies. The major difference between Spark and Hadoop is how they use memory. Hadoop writes intermediate results to disk whereas Spark tries to keep data in memory whenever possible. This makes **Spark faster for many use cases.**

**Hadoop MapReduce** - a system for processing and analyzing large data sets in parallel.

**Hadoop YARN** - a resource manager that schedules jobs across a cluster. The manager keeps track of what computer resources are available and then assigns those resources to specific tasks.

**Hadoop Distributed File System (HDFS)** - a big data storage system that splits data into chunks and stores the chunks across a cluster of computers.

Spark, on the other hand, does not include a file storage system. You can use Spark on top of HDFS but you do not have to. Spark can read in data from other sources as well such as Amazon S3.


**Apache Pig** - a SQL-like language that runs on top of Hadoop MapReduce

**Apache Hive** - another SQL-like interface that runs on top of Hadoop MapReduce

### MapReduce

MapReduce is a programming technique for manipulating large data sets. "Hadoop MapReduce" is a specific implementation of this programming technique.

The technique works by first dividing up a large dataset and distributing the data across a cluster. In the map step, each data is analyzed and converted into a (key, value) pair. Then these key-value pairs are shuffled across the cluster so that all keys are on the same machine. In the reduce step, the values with the same keys are combined together.

### Spark modes

- Local mode
- Standalone 
- YARN
- Mesos: from UC Berkeley Amp lab

### Spark use cases 

- Data analytics
- Machine learning
- Streaming
- Graph Analytics

Spark is meant for big data sets that cannot fit on one computer. For smaller datasets that can fit on a local computer there are alternative tools:
- AWK: a command line tool for manipulating text files
- Python Pydata stack: pandas, matplotlib, numpy, scikit-learn
- R

### Spark Limitations

Spark Streaming’s latency is at least 500 milliseconds since it operates on micro-batches of records, instead of processing one record at a time. Native streaming tools such as Storm, Apex, or Flink can push down this latency value and might be more suitable for low-latency applications. Flink and Apex can be used for batch computation as well, so if you're already using them for stream processing, there's no need to add Spark to your stack of technologies.

Another limitation of Spark is its selection of machine learning algorithms. Currently, Spark only supports algorithms that scale linearly with the input data size. In general, deep learning is not available either, though there are many projects integrate Spark with Tensorflow and other deep learning tools.

### Hadoop versus Spark

The Hadoop ecosystem is a slightly older technology than the Spark ecosystem. In general, Hadoop MapReduce is slower than Spark because Hadoop writes data out to disk during intermediate steps. However, many big companies, such as Facebook and LinkedIn, started using Big Data early and built their infrastructure around the Hadoop ecosystem.

While Spark is great for iterative algorithms, there is not much of a performance boost over Hadoop MapReduce when doing simple counting. Migrating legacy code to Spark, especially on hundreds of nodes that are already in production, might not be worth the cost for the small performance boost.

### Beyond Spark for Storing Big Data 

Spark is not a data storage system, there are a number of tools for this purpose. HBase or Cassandra are popular database storage systems. There are also distributed SQL engines like Impala or Presto. 


### Resources

- [Apache Spark](https://spark.apache.org/)