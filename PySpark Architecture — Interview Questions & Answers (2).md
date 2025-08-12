# PySpark Architecture — Interview Questions & Answers (2)

## Section 1: Fundamentals (1–25)
### 1. What is PySpark?
Python API for Apache Spark that uses Py4J to communicate with the JVM Spark core.

### 2. Key components of PySpark architecture?
Driver Program, Cluster Manager, Executors, Py4J Bridge, RDD/DataFrame API, Catalyst Optimizer, Tungsten Engine.

### 3. Role of the driver in PySpark?
Creates SparkSession, builds DAG, sends tasks to executors, collects results.

### 4. Role of executors?
Run tasks, store data in memory/disk, report status back to driver.

### 5. What is Py4J in PySpark?
Java gateway enabling Python code to call JVM Spark APIs.

### 6. Difference between actions and transformations?
Transformations are lazy (e.g., map, filter), actions trigger execution (e.g., count, collect).

### 7. What is lazy evaluation in PySpark?
Execution is delayed until an action is triggered.

### 8. How does Spark build a DAG?
Each transformation adds nodes and edges representing data flow; Catalyst optimizer optimizes it.

### 9. What is a stage in Spark?
A set of tasks with no shuffle boundary; separated by wide dependencies.

### 10. What is a task in Spark?
Smallest execution unit, processes one partition of data.

### 11. Difference between narrow and wide transformations?
Narrow – no shuffle (map), Wide – shuffle required (groupBy).

### 12. What triggers stage boundaries?
Wide dependencies such as joins, groupBy, repartition.

### 13. Role of Cluster Manager?
Allocates resources (CPU/memory) to Spark applications (YARN, Mesos, Standalone, Kubernetes).

### 14. What is Tungsten execution engine?
Handles memory management, code generation, columnar execution.

### 15. What is Catalyst optimizer?
Query optimizer for DataFrames/SQL, applies rule-based and cost-based optimizations.

### 16. Difference between RDD and DataFrame?
RDD – low-level, no optimizer; DataFrame – high-level, optimized via Catalyst.

### 17. How does Spark handle fault tolerance?
Recomputes lost partitions using RDD lineage.

### 18. What is checkpointing?
Persisting RDD/DataFrame to stable storage to truncate lineage.

### 19. Difference between cache() and persist()?
cache() = MEMORY_ONLY; persist() allows different storage levels.

### 20. How is PySpark code executed on a cluster?
Python → Py4J → JVM driver → Catalyst → Tungsten → Executors.

### 21. Difference between local and cluster mode?
Local – all components on one machine; Cluster – driver and executors on different nodes.

### 22. What is job in Spark?
A set of stages triggered by one action.

### 23. How does Spark decide number of partitions?
spark.default.parallelism for RDDs; spark.sql.shuffle.partitions for DataFrames.

### 24. What is data locality in Spark?
Scheduling tasks close to data to reduce network IO.

### 25. Why is collect() risky?
Pulls all data to driver → OOM risk.

## Section 2: Advanced Concepts (26–50)
### 26. How does Spark handle shuffling?
Writes intermediate data to disk, exchanges data across executors.

### 27. What is broadcast join?
Broadcasts small table to all executors to avoid shuffle.

### 28. How to control broadcast join threshold?
spark.sql.autoBroadcastJoinThreshold (default 10MB).

### 29. Difference between coalesce and repartition?
Coalesce – reduce partitions without shuffle; Repartition – with shuffle.

### 30. How does dynamic allocation work?
Adds/removes executors based on workload.

### 31. What is speculative execution?
Runs backup copies of slow tasks.

### 32. What is skewed data and its impact?
Uneven data distribution → slow tasks.

### 33. How to fix skewed joins?
Salting keys, broadcasting small table, AQE skew join handling.

### 34. What is Adaptive Query Execution (AQE)?
Runtime query plan optimization (partition coalescing, join strategy switch, skew join).

### 35. How does Arrow improve PySpark performance?
Columnar in-memory format for fast Pandas ↔ Spark conversion.

### 36. Difference between Pandas UDF and regular UDF?
Pandas UDF = vectorized batch; Regular UDF = row-by-row.

### 37. How does Spark manage executor memory?
Split into execution and storage memory, dynamically shared.

### 38. How to tune partition size?
2–4 partitions per CPU core; control with spark.sql.shuffle.partitions.

### 39. What is mapPartitions()?
Operates on entire partition, reduces function call overhead.

### 40. How to merge small files in Spark?
coalesce() before writing; use output partitioning.

### 41. Role of external shuffle service?
Serves shuffle files after executor death.

### 42. How to debug PySpark performance issues?
Use Spark UI, .explain(), check shuffle sizes, executor metrics.

### 43. What is Whole Stage Code Generation?
Tungsten feature generating optimized Java bytecode.

### 44. How does Spark handle Python-JVM serialization cost?
Uses Py4J; minimize with Arrow, vectorized operations.

### 45. How to prevent OOM in large joins?
Broadcast small table, increase memory, repartition.

### 46. How to improve join performance?
Partition on join key, filter early, broadcast.

### 47. What is task serialization?
Sending task data and closures from driver to executor.

### 48. What is closure serialization issue?
Python closures capturing large variables → large task size.

### 49. What is lineage graph?
Dependency graph of RDD transformations.

### 50. When to use checkpointing?
Long lineage chains, iterative algorithms.

## Section 3: Scenario-Based (51–70)

### 51. Job takes long time due to shuffle. How to fix?
Increase partitions, use broadcast, filter early.

### 52. Executor lost during processing. Impact?
Tasks rerun on other executors using lineage.

### 53. Too many small files output. Fix?
Coalesce/repartition before write.

### 54. Python UDF is slow. Fix?
Replace with built-in functions or Pandas UDF.

### 55. Out-of-memory in executor during aggregation. Fix?
Increase executor memory, reduce partition size, spill to disk.

### 56. Job stuck due to straggler task. Fix?
Enable speculative execution.

### 57. Skewed groupBy causing slow stage. Fix?
Key salting, AQE skew join.

### 58. High GC time in executors. Fix?
Reduce memory pressure, repartition, use off-heap storage.

### 59. Local mode job too slow. Why?
Limited cores; switch to cluster mode.

### 60. Multiple small shuffles. Fix?
Combine transformations, repartition strategically.

### 61. Need to speed up Pandas ↔ Spark conversion.
Enable Arrow (spark.sql.execution.arrow.enabled=true).

### 62. DataFrame .count() is slow. Why?
Scans entire dataset; use metadata if available.

### 63. Large dataset join but only few keys needed.
Filter before join.

### 64. Why use mapPartitions() for external DB writes?
Fewer connection creations per partition.

### 65. Why not always cache everything?
Memory waste, eviction of useful data.

### 66. Why job runs faster second time?
Cached/shuffle data reuse.

### 67. Why .explain() shows more stages than expected?
Extra shuffles due to repartitioning, wide dependencies.

### 68. Why .collect() can crash driver but not executors?
Driver memory is separate; executors spill to disk.

### 69. Why broadcast join sometimes slower?
If broadcast dataset is large, serialization overhead.

### 70. Why tasks fail with “Task not serializable” error?
Python closure or object not serializable by Spark.

