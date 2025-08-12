# PySpark Architecture — Interview Questions & Answers (1)

### 1. What are the key components of PySpark architecture?
Answer:
PySpark architecture follows the Spark architecture but adds a Python API layer. The main components are:
- Driver Program – Runs the PySpark application, creates the SparkContext or SparkSession. In Python, this’s the process you run locally or on a cluster edge node.
- Cluster Manager – YARN, Mesos, or Standalone. Allocates resources (CPU, memory) to Spark executors.
- Executors – JVM processes running on worker nodes to execute tasks. They hold data in memory or disk.
- Py4J Bridge – Allows Python code to interact with the JVM-based Spark core.
- RDD/DataFrame API – User-facing abstraction for distributed datasets.
- Catalyst Optimizer – Optimizes logical and physical query plans for DataFrames/Datasets.
- Tungsten Execution Engine – Handles memory management and code generation for efficient execution.

### 2. How does PySpark code written in Python get executed in the JVM-based Spark engine?
Answer:
- PySpark uses Py4J to communicate between the Python process and the JVM process.
- When you write PySpark code, the Python driver translates API calls into Spark Java objects via Py4J.
- These Java objects are processed by Spark’s core engine (JVM-based), and execution plans are generated.
- The JVM sends execution instructions to executors.
- Data processed on executors is serialized (e.g., via Arrow, Pickle) before being returned to the Python process if needed.

### 3. What is the execution flow of a PySpark job?
Answer:
- User Code – You write transformations and actions in Python.
- DAG Creation – Spark builds a Directed Acyclic Graph of transformations.
- Logical Plan – Catalyst optimizer creates an optimized logical plan.
- Physical Plan – Spark generates a physical plan with stages and tasks.
- Task Scheduling – Tasks are distributed to executors by the cluster manager.
- Execution – Executors process tasks and return results.
- Result Collection – Data is sent back to the driver if required (e.g., collect()).

### 4. What are the limitations of PySpark due to the Py4J bridge?
Answer:
- Serialization Overhead – Data must be serialized/deserialized between Python and JVM.
- Garbage Collection – Python and JVM have separate memory spaces; inefficient handling may cause high GC pauses.
- No Direct JVM Access – Some low-level optimizations available in Scala/Java Spark aren’t directly exposed in Python.
- Performance – Python UDFs are slower because they require crossing the Py4J boundary for each row unless using Pandas UDFs with Apache Arrow.

### 5. How do you optimize performance in PySpark given the Python-JVM boundary?
Answer:
- Use built-in SQL/DataFrame functions instead of Python UDFs.
- If UDFs are needed, prefer Pandas UDFs (vectorized) with Arrow.
- Minimize .collect() and .toPandas() calls.
- Push computation as close to the source as possible.
- Cache/repartition wisely to reduce shuffles.
- Tune spark.sql.shuffle.partitions and executor memory.

### 6. What’s the role of the Catalyst optimizer in PySpark?
Answer:
- Catalyst Optimizer is part of Spark SQL, responsible for:
- Parsing the DataFrame API or SQL query into a logical plan.
- Applying rule-based optimizations (e.g., predicate pushdown, constant folding).
- Generating a physical plan with optimized join strategies, partitions, and stages.
- These optimizations work regardless of whether the code is written in Python, Scala, or Java.

### 7. What’s the difference between RDD and DataFrame execution in PySpark?
Answer:
- RDD – No Catalyst optimizer; execution plan is more manual; transformations are lower-level.
- DataFrame – Catalyst optimizer and Tungsten engine optimize execution; better for performance.
- For most use cases, DataFrames are faster due to code generation and vectorized execution.

### 8. How does PySpark handle data shuffling?
Answer:
- Shuffling happens when data needs to be redistributed across partitions (e.g., after groupByKey, joins).
- It involves writing intermediate data to disk, sending it over the network, and reading it back.
- Expensive operation; to minimize shuffles, use map-side combines, repartition carefully, and use broadcast() joins for small datasets.

### 9. What’s the role of Arrow in PySpark?
Answer:
- Apache Arrow provides an in-memory columnar data format for efficient serialization.
- PySpark uses Arrow to speed up Pandas DataFrame ↔ Spark DataFrame conversion.
- Pandas UDFs (vectorized UDFs) use Arrow to process batches of rows at once, avoiding Python-JVM per-row calls.

### 10. How does PySpark work in cluster mode vs local mode?
Answer:
- Local mode – Driver and executors run in the same machine; good for testing.
- Cluster mode – Driver runs on a cluster node (or client machine in client mode); executors run on worker nodes.
- In client mode, the driver runs where you submit the job.
- In cluster mode, the driver runs inside the cluster, and results are returned to the submission client.

### 11. How are jobs, stages, and tasks structured in PySpark?
Answer:
- Job – Triggered by an action (collect(), count(), save()).
- Stage – A set of tasks without shuffle dependencies (either narrow or wide). Stages are separated by shuffle boundaries.
- Task – The smallest execution unit, runs on one partition in an executor.
  Flow: Job → Multiple Stages → Multiple Tasks per Stage.

### 12. What’s the difference between a narrow dependency and a wide dependency in Spark?
Answer:
- Narrow Dependency – Child RDD partition depends on a small, fixed set of parent partitions (e.g., map, filter). No shuffle needed.
- Wide Dependency – Child RDD partition depends on many partitions (e.g., groupByKey, reduceByKey). Requires shuffle.
  Narrow = faster; Wide = expensive.

### 13. How does Spark determine stage boundaries?
Answer:
- Stage boundaries occur when there’s a wide dependency that requires a shuffle. For example:
```
df.groupBy("id").count()
```
- This causes Spark to split the job into two stages:
  Stage 1: Map-side processing.
  Stage 2: Reduce-side aggregation after shuffle.

### 14. How can you view the DAG and execution plan in PySpark?
Answer:
- Use df.explain(True) for a detailed execution plan (logical + physical).
- Use Spark UI (usually at http://<driver-host>:4040) → DAG Visualization tab.
- This helps identify shuffle boundaries and bottlenecks.

### 15. What are broadcast variables and when should you use them?
Answer:
- Broadcast variables send a read-only copy of data to all executors.
- Useful for small lookup tables in joins to avoid shuffling large datasets.
  ```
  broadcast_var = spark.sparkContext.broadcast(my_dict)
  ```
- Use broadcast() joins for small dimension tables.

### 16. What are accumulators in PySpark?
Answer:
- Write-only shared variables for aggregating information across executors to the driver.
- Commonly used for debugging or counters.
```
accum = sc.accumulator(0)
```
- Not guaranteed for exactly-once semantics due to task retries.

### 17. How does PySpark optimize joins?
Answer:
- Broadcast Join – Avoids shuffle by broadcasting small dataset to all executors.
- Sort-Merge Join – Default for large datasets; requires both sides to be sorted.
- Shuffle Hash Join – Used when one side fits into memory after shuffle.
  Tuning spark.sql.autoBroadcastJoinThreshold controls broadcast behavior.

### 18. What’s the difference between coalesce() and repartition()?
Answer:
- repartition(n) – Increases or decreases partitions with a full shuffle.
- coalesce(n) – Reduces partitions without shuffle (only merges).
  Use coalesce() when decreasing partitions to avoid shuffle.

### 19. How does PySpark manage memory for execution and storage?
Answer:
- Memory is split into:
  - Execution memory – For shuffles, joins, aggregations.
  - Storage memory – For caching RDDs/DataFrames.
- Managed dynamically; unused storage can be borrowed by execution and vice versa.
- Controlled via spark.memory.fraction and spark.memory.storageFraction.

### 20. How do you persist data in PySpark, and what’s the difference between cache() and persist()?
Answer:
- cache() = persist(StorageLevel.MEMORY_ONLY).
- persist() allows custom storage levels (e.g., MEMORY_AND_DISK).
- Always unpersist when data is no longer needed to free memory.

### 21. What’s the difference between client mode and cluster mode in Spark?
Answer:
- Client Mode – Driver runs on the machine where you submit the job.
- Cluster Mode – Driver runs inside the cluster on a worker node.
  Cluster mode is better for production to avoid driver shutdown when the client disconnects.

### 22. How can you tune the number of partitions in PySpark?
Answer:
- Control initial partitions via spark.default.parallelism and spark.sql.shuffle.partitions.
- Repartition based on dataset size, cluster resources, and avoiding too small/large partitions. Rule of thumb: 2–4 partitions per CPU core.

### 23. How does PySpark handle fault tolerance?
Answer:
- RDDs have lineage: Spark rebuilds lost partitions by re-running the transformations.
- Checkpointing to HDFS can be used to truncate long lineage chains.

### 24. How does speculative execution work in PySpark?
Answer:
- Detects slow-running tasks (stragglers) and runs backup copies on other nodes.
- First completed task result is used.
- Controlled by spark.speculation and spark.speculation.quantile.

### 25. What’s the difference between Pandas UDFs and regular UDFs in PySpark?
Answer:
- Regular UDFs – Row-by-row processing; high overhead due to Python-JVM calls.
- Pandas UDFs – Vectorized batch processing with Apache Arrow; much faster.

### 26. How does PySpark handle data locality?
Answer:
- Spark tries to schedule tasks close to where the data resides (node or rack locality).
- Improves performance by reducing network IO.

### 27. What’s the difference between narrow and wide transformations in terms of fault tolerance?
Answer:
- Narrow transformations can recompute lost partitions without touching other partitions.
- Wide transformations require re-reading/shuffling multiple partitions.

### 28. How does the Tungsten execution engine improve performance?
Answer:
- Uses off-heap memory management to reduce GC pressure.
- Generates optimized bytecode at runtime (whole-stage codegen).
- Enables cache-friendly columnar storage formats.

### 29. How does PySpark handle skewed data in joins?
Answer:
- Skewed keys cause some tasks to take much longer.
- Mitigation strategies:
  - Salting keys (adding random prefixes).
  - Broadcasting small table.
  - Using spark.sql.adaptive.skewJoin.enabled.
 
### 30. How does Adaptive Query Execution (AQE) work in PySpark?
Answer:
- AQE dynamically optimizes query plans at runtime based on actual stats.
- Features:
  - Dynamically coalescing shuffle partitions.
  - Switching join strategies based on runtime sizes.
  - Handling skewed joins.
    Enabled via spark.sql.adaptive.enabled=true.

### 31. What happens in PySpark when you call an action after multiple transformations?
Answer:
- Transformations are lazy, meaning Spark doesn’t execute them immediately.
- When an action (count, collect, save) is called:
  - Spark builds a logical plan from the transformations.
  - Catalyst optimizer optimizes the plan.
  - A physical execution plan is created, split into stages.
  - Tasks are executed across executors.
 
### 32. How does Spark decide the number of shuffle partitions?
Answer:
- Controlled by spark.sql.shuffle.partitions (default = 200).
- Affects operations like joins, groupBy, and aggregations.
- Too high → unnecessary overhead. Too low → skew and OOM errors.
- Best practice: tune based on data size and cluster resources.

### 33. How is PySpark different from running pure Python in terms of execution model?
Answer:
- Python executes sequentially, whereas PySpark distributes computation across multiple executors.
- PySpark’s Python code does not execute in parallel by default; instead, it sends execution instructions to Spark’s JVM engine.
- Real computation happens in JVM executors, not Python processes (except UDFs).

### 34. What’s the role of serialization in PySpark performance?
Answer:
- Data sent between driver ↔ executor or Python ↔ JVM must be serialized.
- Common serializers:
  - Pickle (default, slower)
  - CloudPickle (supports more objects)
  - Kryo (for JVM serialization, faster)
- Use Arrow for vectorized Pandas UDFs to avoid row-by-row serialization.

### 35. Why is collect() dangerous in PySpark?
Answer:
- Brings all data from executors to the driver.
- Risks:
  - Out-of-memory (OOM) on driver.
  - Network congestion.
- Alternative: use take(n), limit(n), or foreach.

### 36. How do you handle out-of-memory errors in PySpark executors?
Answer:
- Increase executor memory: --executor-memory or spark.executor.memory.
- Reduce shuffle size by increasing partitions.
- Avoid caching unnecessary datasets.
- Use persist(StorageLevel.MEMORY_AND_DISK) instead of memory-only caching.

### 37. How does Spark UI help in debugging performance issues?
Answer:
- Shows DAG visualization for job stages.
- Displays shuffle read/write sizes to detect bottlenecks.
- Identifies skewed tasks and executor failures.
- Memory and GC statistics for tuning.

### 38. How do you reduce shuffles in PySpark joins?
Answer:
- Use broadcast() for small datasets.
- Repartition both datasets on join key before joining.
- Use bucketing in table design.
- Filter early before joining.

### 39. What’s the difference between checkpointing and caching in PySpark?
Answer:
- Caching – Keeps RDD/DataFrame in memory for reuse. Still depends on lineage for recomputation if lost.
- Checkpointing – Saves data to HDFS; truncates lineage to avoid long recomputation chains. Used for long-running jobs.

### 40. How do you debug Python UDF performance in PySpark?
Answer:
- Enable Arrow to vectorize (spark.sql.execution.arrow.enabled=true).
- Check if the logic can be expressed using built-in Spark SQL functions.
- Profile with .explain() to see if UDF is blocking optimizations.

### 41. How does Spark handle large shuffle files?
Answer:
- Writes intermediate data to disk in shuffle spill files.
- Merges small shuffle files to reduce file system overhead.
- Uses external shuffle service to serve files after executor death.

### 42. How do you handle executor loss in PySpark?
Answer:
- Spark re-schedules lost tasks to other executors.
- RDD lineage allows recomputation.
- If data was cached only in the lost executor, recomputation will happen from source.

### 43. What’s the role of speculative execution in slow job recovery?
Answer:
- Runs backup copies of slow tasks (stragglers).
- First to finish wins, reducing total job time.
- Useful in uneven cluster performance scenarios.

### 44. What’s the difference between DataFrame API and SQL API in PySpark performance?
Answer:
- Both use the Catalyst optimizer internally, so performance is similar.
- DataFrame API offers type safety (in Scala/Java, not Python).
- SQL API is easier for complex query logic.

### 45. How do you optimize partition size for reading from Parquet in PySpark?
Answer:
- Adjust spark.sql.files.maxPartitionBytes to control partition size when reading.
- Combine small files using coalesce() before processing.
- Use predicate pushdown to avoid reading unnecessary files.

### 46. What’s the difference between mapPartitions() and map() in PySpark?
Answer:
- map() – Operates on each row individually.
- mapPartitions() – Operates on entire partition at once; fewer function calls → better performance.

### 47. How does PySpark handle heterogeneous cluster resources?
Answer:
- By default, tasks are scheduled to available executors.
- If one node is slower, speculative execution can help.
- No built-in resource balancing beyond partition distribution.

### 48. How do you handle skewed data in aggregations?
Answer:
- Salting keys before aggregation.
- Splitting large keys into smaller chunks.
- Using approx_count_distinct instead of exact counts.

### 49. How does dynamic allocation work in PySpark?
Answer:
- Executors are added/removed based on workload.
- Controlled by spark.dynamicAllocation.enabled.
- Needs external shuffle service enabled.

### 50. How do you run multiple PySpark jobs concurrently without contention?
Answer:
- Use different queues in YARN with capacity limits.
- Tune spark.sql.shuffle.partitions to avoid shuffle saturation.
- Limit concurrent jobs to available cores and memory.

