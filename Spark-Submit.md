# Distribution of Executors, Cores and Memory for a Spark Application running in Yarn:
## Following list captures some recommendations to keep in mind while configuring them:
1. Hadoop/Yarn/OS Deamons: When we run spark application using a cluster manager like Yarn, there’ll be several daemons that’ll run in the background like NameNode, Secondary NameNode, DataNode, JobTracker and TaskTracker. So, while specifying num-executors, we need to make sure that we leave aside enough cores (~1 core per node) for these daemons to run smoothly.
2. Yarn ApplicationMaster (AM): ApplicationMaster is responsible for negotiating resources from the ResourceManager and working with the NodeManagers to execute and monitor the containers and their resource consumption. If we are running spark on yarn, then we need to budget in the resources that AM would need (~1024MB and 1 Executor).
3. HDFS Throughput: HDFS client has trouble with tons of concurrent threads. It was observed that HDFS achieves full write throughput with ~5 tasks per executor . So it’s good to keep the number of cores per executor below that number.
4. MemoryOverhead: Following picture depicts spark-yarn-memory-usage.

![image](https://user-images.githubusercontent.com/58762764/127876430-d560bd07-7ff6-438c-9508-3f31ea870c0c.png)

### Two things to make note of from this picture:
```
Full memory requested to yarn per executor = spark-executor-memory + spark.yarn.executor.memoryOverhead
spark.yarn.executor.memoryOverhead = Max(384MB, 7% of spark.executor-memory)
```

So, if we request 20GB per executor, AM will actually get 20GB + memoryOverhead = 20 + 7% of 20GB = ~23GB memory for us.
* Running executors with too much memory often results in excessive garbage collection delays.
* Running tiny executors (with a single core and just enough memory needed to run a single task, for example) throws away the benefits that come from running multiple tasks in a single JVM.
