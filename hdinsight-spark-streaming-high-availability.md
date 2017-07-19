---
title: Creating highly available Spark Streaming jobs in YARN - Azure HDInsight | Microsoft Docs
description: ''
services: hdinsight
documentationcenter: ''

tags: azure-portal
keywords: spark, streaming, YARN
---
# Creating highly available Spark Streaming jobs in YARN

## Understanding Spark Streaming Jobs

![Spark Streaming](./media/hdinsight-spark-streaming-high-availability/spark-streaming.png)

Spark Streaming enables you to implement scalable, high-throughput, fault-tolerant applications for the processing of data streams. Running Spark Streaming applications on a HDInsight Spark cluster, you can connect them to process data from a variety of sources such as Azure Event Hubs, Azure IoT Hub, Kafka, Flume, Twitter, ZeroMQ, raw TCP sockets or even by monitoring the HDFS filesystem for changes.  

Spark Streaming creates long-running jobs during which you are able to apply transformations to the data and then push out the results to filesystems, databases, dashboards and the console. It uses micro-batches to processes data, by waiting to collect a time-defined batch of events (usually configred in the range of less than a second to a few seconds), before it sends the batch of events on for processing. It supports fault tolerance with the guarantee that any given event would be processed exactly once, even in the face of a node failure.

### Understanding DStreams

![Spark DStream](./media/hdinsight-spark-streaming-high-availability/DStream.png)

 Spark Streaming represents a continous stream of data using a discretized stream or DStream. This DStream can be created from input sources like Event Hubs or Kafka, or by applying transformation on another DStream. When an event arrives at your Spark Streaming application, this event is stored in a reliable way- it is replicated so that multiple nodes have a copy of it. This ensures that the failure of any single node will not result in the loss of your event. 
 
 Spark core uses RDDs (resiliant distributed datasets). RDDs distribute data across multiple nodes in the cluster, where each node generally maintains its data completely in-memory for best performance. Each RDD represents events collected over some user defined timeframe called the batch interval. Every time the batch interval elapses a new RDD is produced that contains all the data in the interval of time that just completed. This continuous set of RDDs are collected into a DStream.  A Spark Streaming application processes micro-batches containing events, and ultimately acts on the data stored in the RDD each batch contains. 

### Understanding Spark Structured Streaming Jobs

![Spark Structured Streaming](./media/hdinsight-spark-streaming-high-availability/structured-streaming.png)

Spark Structured Streaming was introduced in Spark 2.0.  It is a an analytic engine for streaming structured data.  Spark Structured Streaming shares the same set of APIs with the SparkSQL batching engine.  As with Spark Streaming, Spark Structured Streaming provides the ability to run computation over continuously arriving data and uses a micro-batches streaming model. Spark Structured Streaming represents a stream of data as an unbounded Input Table (heightwise) - in other words the table continues to grow as new data arrives. This Input Table is continously processed by a long running query, and the results flushed out to an Output Table. 

In Structured Streaming data arrives at the system and is immediately ingested into the Input Table. You write queries that perform operations against this Input Table. The query output yields another table, called the Results Table. The Results Table contains results of your query from which you draw any data you would send to an external datastore (such a relational database). The timing of when data is processed from the Input Table is controlled by the trigger interval. By default Structured Streaming tries to process the data as soon as it arrives. However, you can also configure the trigger to run on a longer interval, so that the streaming data is processed according to time-based batches. The output mode controls the data in the Results Tables. This data may be completely refreshed everytime there is new data so that it includes all of the output data since the streaming query began (called `complete mode`), or it may only contain just the data that is new since the last time the query was processed (called `append mode`).

---
## Creating Fault Tolerant Spark Streaming Jobs

In order to be able to create a highly-available environment for your Spark Streaming Jobs, you must first start by architecting your individual jobs for recovery in the event of failure.  This is also called creating fault-tolerant jobs.  This section covers relavent techniques. RDDs have a number of properties that aid the ability to create highly-available and fault tolerant Spark Streaming Jobs.  These properties are listed below:

* Batches of input data stored in RDDs as a DStream are autopmatically replicated in memory for fault-tolerance
* Data lost due to worker failure can be recomputed from replicated input data on different workers, as long as those worker nodes are available
* Fast Fault Recovery can occur within 1 second, as recovery from faults/stragglers happens via computation in memory

### Achieving Exactly Once Semantics with Spark Streaming

To achieve exactly once processing of an event by a Spark Streaming application, you need to consider how all of the points of failure restart after having an issue and how you can avoid data loss. Consider that in a Spark Streaming application you have an input source, a driver process that manages the long running job, one or more receiver processes that pull data from the input source, and tasks that apply the processing and push the results to an output sink. To achieve exactly once semantics means ensuring that no data is lost at any point and that the processing is restartable, regadless of where the failure occurs. The following describes each of the key components of achieving exactly once semantics.

### Replayable Sources & Reliable Receivers

The source your Spark Streaming application is reading your events from must be replayable. This means that it should be possible to ask the source to provide the message again in cases where the message was retrieved but then the system failed before it could be persisted or processed. In Azure  both Azure Event Hubs and Kafka on HDInsight provide replayable sources. An even simpler example of a replayable source is a fault-tolerant file system like HDFS, Azure Storage blobs or Azure Data Lake Store where all the data can be kept in perpetuity (and you can re-read it in its entirety at any point). In Spark Streaming, sources like Event Hubs and Kafka have reliable receivers, meaning they keep track of their progress reading through the source and persist it to fault-tolerant storage (either within ZooKeeper or in Spark Streaming checkpoints written to HDFS). That way, if a receiver fails and is later restarted it can pick up where it left off.    

#### Use the Write Ahead Log
Spark Streaming supports the use of a Write Ahead Log, where any event received from a source is first written to Spark's checkpoint directory in fault-tolerant storage (in Azure this is HDFS backed by either Azure Storage or Azure Data Lake Store) before being stored in an RDD by the receiver running within a worker. In your Spark Streaming application, the Write Ahead Log is enabled for all receivers by setting the `spark.streaming.receiver.writeAheadLog.enable` configuration to `true`. The Write Ahead Log provides fault-tolerance for failures of both the driver and the executors.  

In the case of workers running tasks against the event data, consider that following the insertion into the Write Ahead Log, the event is inserted by the receiver into an RDD, which by definition is both replicated and distributed across multiple workers. Should the task fail because the worker running it crashed, the task will simply be restarted on another worker that has a replica of the event data, so the event is not lost. 

#### Use Checkpoints
The drivers need to be restartable. If the driver running your Spark Streaming application crashes, it takes down with it all executors running receivers, tasks and any RDD's storing event data. This surfaces two considerations- first you need to be able to save the progress of the job so you can resume it later. This is accomplished by checkpointing the Directed Acyclic Graph (DAG) of the DStream periodically to fault-tolerant storage. This metadata includes the configuration used to create the streaming application, the operations that define the application, and any batches that are queued but not yet completed. This will enable a failed driver to be restarted from the checkpoint information. When it restarts, it will launch new receivers that themselves recover the event data back into RDD's from the Write Ahead Log. Checkpoints are enabled in Spark Streaming in two steps. On the StreamingContext object you configure the path in storage to where the checkpoints are stored:
```scala
    val ssc = new StreamingContext(spark, Seconds(1))
    ssc.checkpoint("/path/to/checkpoints)
```
In HDInsight, these checkpoints should be saved to your default storage attached to your cluster (either Azure Storage or Azure Data Lake Store). Next, you need to specify a checkpoint interval (in seconds) on the DStream that controls how often any state data (e.g., state derived from the input event) is persisted to storage. Persisting state data this way can reduce the computation needed when rebuilding the state from the source event. 
```scala
    val lines = ssc.socketTextStream("hostname", 9999)
    lines.checkpoint(30)
    ssc.start()
    ssc.awaitTermination()
```

####  Use Idempotent Sinks

The destination to which your job writes results must be able to handle the situation where it is given the same result more than once. It must be able to detect such duplicate results and ignore them. You can achieve idempotent sinks by implementing logic that first checks for the existence of the result in the datastore. 
* If the result already exists, then the write should appear to succeed from the perspective of your Spark job, but in reality your data store ignored the duplicate data. 
* If the result does not exist, then it should insert the new result into its storage. One example of this is to use a stored procedure with Azure SQL Database that is used to insert events into a table. 
* When the stored procedure is invoked, it first looks up the event by key fields and only if none are found is the record inserted into the table. Another example is to use a partitioned file system, like Azure Storage blobs or Azure Data Lake store. In this case your sink logic does not need to check for the existence of a file. If the file representing the event exists, it is simply overwritten with the same data. Otherwise, a new file is created at the computed path. In the end idempotent sinks should support being called multiple times with the same data and no change of state should result. 

--- 

## About Spark Streaming and YARN

![YARN Architecture](./media/hdinsight-spark-streaming-high-availability/yarn-arch.png)

In HDInsight cluster work is coordinated by YARN.  Architecting for high availability for Spark Streaming has to include not only techniques for Spark Streaming, but also for YARN components.  An example configuration using YARN is shown above.  There are a number of considerations for this configuration. 

### Planning for failure

Considering which components could fail, namely an executor or a driver, is a first step in augmenting your HDInsight cluster YARN configuration for high-availability.  Also, certain Spark Streaming job failures may include data guarantee requirements which needs additonal configurations and setups.  For example, a streaming application may have the business requirement for ZERO data loss guarantees in spite of any type of error that could occur in the hosting streaming system or HDInsight cluster.

If an **executor** fails, then tasks and receivers are restarted by Spark automatically, there is no configuration change needed.  Importantly, if a **driver** fails, then all of its associated executors fail.  Also all computation and received blocks are lost.  Use DStream Checkpointing (discussed in an earlier section of this article) to recover from driver failure.  As mentioned previously, DStream Checkpointing periodically saves the DAG of DStreams to fault-tolerant storage (such as Azure BLOBs).  Checkpointing allows for the failed driver to be restarted using checkpoint information.  This driver restart will launch new executors and will also restart receivers.

To recover drivers w/DStream Checkpointing
* Configure automatic driver restart on the YARN by updating the configuration `yarn.resourcemanager.am.max-attempts` setting
* Set a checkpoint directory in a HDFS-compatible file system by `streamingContext.checkpoint(hdfsDirectory)`
* Restructure source code to use checkpoints for recovery (example function shown below for setup code)
* Configure to recover lost data by enabling WAL via `sparkConf.set("spark.streaming.receiver.writeAheadLog.enable","true")` and
disabling in-memory replication via `StorageLevel.MEMORY_AND_DISK_SER` for input DStreams

```scala
    def creatingFunc() : StreamingContext = {
        val context = new StreamingContext(...)
        val lines = KafkaUtils.createStream(...)
        val words = lines.flatMap(...)
        ...
        context.checkpoint(hdfsDir)
    }

    val context = StreamingContext.getOrCreate(hdfsDir, creatingFund)
    conext.start()
```

To summarize, using checkpointing + WAL + reliable receivers, you will be able to deliver "at least once" data recovery
* Exactly once, as long as received data is not lost and if outputs are idempotent or transactional
* Exactly once (alternative) via new Kafka Direct approach - uses Kafka as replicated log, does not use receivers or WALs

---

### Common Mistakes in High Availability and Mitigation Steps

* Not monitoring and managing your streaming jobs - It's much more difficult to monitor streaming jobs than batch jobs. Spark streaming jobs are typically long-running.  Also YARN doesn't aggregate logs until a job finishes.  Spark checkpoints can't survive app or Spark upgrades.  Need to clear checkpoint directory during upgrade.  Configure your YARN Cluster mode to run drivers even if client fails.  Set up automatic restart on driver, update spark configuration parameters to do this as shown below.

```
    spark.yarn.maxAppAttempts = 2
    spark.yarn.am.attemptFailuresValidityInterval=1h
```
* In order to work with enhanced metrics for monitoring, here are some considerations.  Spark Streaming UI AND Spark have a configurable metrics system, or you can also use additional libraries, such as Graphite/Grafana to download dashboard metrics such as 'num records processed', 'memory/GC usage on driver & executors', 'total delay', 'utilization of the cluster' and others...
In Structured Streaming (2.1 or greater only) you can use `StreamingQueryListener` to gather additional metrics. 

* Not segmenting long-running jobs.  When a Spark Streaming application is submitted to the cluster, the YARN queue where the job runs must be defined. You can use a [YARN Capacity Scheduler](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/CapacityScheduler.html) to submitting long-running jobs to separate queue.

* Not shutting down your streaming application  gracefully - If your offsets are known and state stored is externally then you can programmatically stop your streaming application at the appropriate place. One technique is to use 'thread hooks' in Spark, by checking for external flag every n seconds -or- use by using a marker file. You can 'touch' this file when starting the 
application on HDFS and then remove the file when you want to stop. If using this method,  use a separate thread in Spark application, which calls code similiar that that shown below.
```scala
    streamingContext.stop(stopSparkContext = true, stopGracefully = true)
    //store offsets externally in an external database, to be able to recover on restart
```

---

## Conclusion
This article provided a series of actions that you can to take to run Spark Streaming jobs in a fault-tolerant and highly-available way on a YARN cluster on HDInsight. These includes making cluster configuration changes for both YARN and Spark Streaming, for example to enable checkpointing and use the write-ahead log, and detailed in which scenarios the various activities should be used.

- [Spark Streaming Overview](hdinsight-spark-streaming-overview) 
- [Long-running Spark Streaming Jobs on YARN](http://mkuthan.github.io/blog/2016/09/30/spark-streaming-on-yarn/) 
- [Structured Streaming: Fault Tolerant Semantics](http://spark.apache.org/docs/2.1.0/structured-streaming-programming-guide.html#fault-tolerance-semantics)
- [Discretized Streams: A Fault-Tolerant Model for Scalable Stream Processing](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2012/EECS-2012-259.pdf)
