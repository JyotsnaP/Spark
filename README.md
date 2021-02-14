# Spark 101

## Spark

### Advantages of spark
---
- SPEED
- EASE OF USE
- A UNIFIED ENGINE

### Differences b/w traditional hadoop map reduce and spark
---

| FEATURE 		 	 | HADOOP MAP REDUCE 		 	 | SPARK   						|			  
|:-------------------------- |:--------------------------|:--------------------------|
| `SPEED`      			 	 | Faster than single machine 	 | 100X faster than MR   |   			 	 
| `WRITTEN IN`    	 | Java      						 	  | Scala      			 |	 
| `EASE OF USE` 	 | Complex and lengthy 						 	  | Simple and crisp    |  			 	 
| `DATA PROCESSING` 	 	 | Batch processing 							 	 | Batch/Real time/iterative/graph |     			 	 
| `CACHING` 	 	 | Does not support caching, writes data to disk back and forth creating an I/O bottleneck    	  | Caches the data in-memory, enhancing the performance |     			 	 

## Spark and Distributed Computing jargons

- Partioned data
- RDDS
- Dataframes
- Fault Tolerance
- Lineage Graph
- Lazy evaluation
- In memory computation
- Transformations
- Actions
- Spark Job
---

### Partitioned Data, RDD and Dataframes
A partition is nothing but an atomic chunk of data that is stored on a node in a cluster. Partitions primarily exist to facilitate parallelism in Apache Spark. **RDDS** in Apache Spark are a collection of partitions. An **RDD** is the fundamental data structure and building block of Apache Spark.

**RDDS** are:
  - Resilient Distributed Dataset
  - an immutable collection of objects
  - logically paritioned


**Etymology**:
- **Resilient**: Due to their immutability and lineage graphs (DAG)
- **Distributed**: Due to inbuilting partitioning
- **Data**: Oh well, they do hold some data !!


One thing to note about an RDD is that it does not have a schema. They are not stored in an columnar structure or tabular. Data is just stored in them row-by-row and are displayed similar to a list (Like a list of rows - Ex -> [row(...)])

| FEATURE 		 	 | RDDS 		 	 | SPARK DATAFRAME   				|					  
|:-------------------------- |:-------------------------|:-------------------------|
| `STORAGE`      			 	 | Not stored in columnar format. They are stored as list of rows 	 | They are stored in columnar format  |  
| `SCHEMA`      			 	 | No schema 	 | Has all the features of an RDD but also has a schema. This is the my chice of data structure while coding in Pyspark
**Dataframe - has a schema**
![DF_has_schema](https://github.com/JyotsnaP/Spark/blob/master/Images/df_has_schema.png)
--
**RDD - doesn't have a schema**

![RDD_does_not_schema](https://github.com/JyotsnaP/Spark/blob/master/Images/rdd_does_not_schema.png)
--
**RDD - is a list**

![RDD_is_a_list_of_rows](https://github.com/JyotsnaP/Spark/blob/master/Images/rdd_is_a_list_of_rows.png)

### Fault tolerance and Lineage Graph
As the name suggests, it is basically a mechanism in which Apache Spark is able to tolerate some amounts of faults. What this means is that the system must be ablt to gracefully continue work properly in the event of a failure without having to throw it's hands up in the air. A failure could be anything, like a node went down or any networking disturbances. 

Fault tolerance in Apache Spark revolves around the concept of RDDs. This feature of self-recovery is one of the defining powers of Apache Spark. For this Apache Spark usings a Lineage Graph/**D**irected **A**cyclic **G**raph. This is nothing but a series of logical steps that constitute the program itself; this is what I'd like to call a "logiacal execution plan". So if a node was crashes, the cluster manager find outs which node that was - and gets all the information about what that node was supposed to do in the lineage graph and assign it to another node to continue processing at the same place and time. This new node will operate on the same partition of the RDD. And because of this new node, there is **no data loss**

A Dag looks something like this: 
![dag](https://github.com/JyotsnaP/Spark/blob/master/Images/dag.png)

The DAG can be found in the **Spark History Server** on the **AWS EMR** console 
![aws_console_spark_history](https://github.com/JyotsnaP/Spark/blob/master/Images/aws_console_spark_history.png)

### Lazy evaluation
####* Being lazy in general has a negative connotation, but not in the context of Spark *

Going by its name, it is safe to say that Spark might be lazy, but extremely efficient nevertheless. It will not start execution unless an action is triggered. Transformations are lazy by nature - Spark keeps track of what transformation is called on which record(using the DAG) and will execute them only when an action is called on the data(for ex, printing the top 5 lines of the dataset). Hence, Spark ensures that data is not loaded and worked upon until and unless it is absolutely needed.

An analogy for this is like when a teacher decides to ask a question in the class. It is also the rule of the class not to answer in mass, and only answer when specifically pointed to and asked. Now lets say the teacher asks a bunch of students what is `5 times 9`. If `Apache Spark` were a student in that class, he/she would use their brain to compute `5 times 9` only when the teacher says `Apache Spark, what is the answer` - Note that `what is the answer` - is equivalent of an `action`

![lazy_evaluation](https://github.com/JyotsnaP/Spark/blob/master/Images/lazy_evaluation.png)

### In memory computation


### Transformations
Spark Transformations is basically a function or set of functions performed on an RDD to get a new RDD. Note here that transformations return new RDDs since RDDs are immutable. Transformations are lazy in nature, what this means is that a tranformation gets **"executed"** only when an **"action"** is called on it. Two of the most basic transformations are: map() and filter()

There are `two types` of transformations: 
 - **Narrow Transformation**
  All the elements required to compute the records in a single partition reside in a single partition of the parent RDD. This means that a subset of the partition can be used to calculate whatever result we want. 
  Ex. 
  map(),mapPartition(),flatMap(),filter(),union()

  One way to look at this is: 

| PARTITION| Item| Cost| Store |
|:------------------ |:------------------------------|:-----------------------------|
| `1`| Tiramisu 	 	 | 10$		 |	Safeway						|
| `1`  | Doritoes 	 	 | 5$		 |	Costco						|
| `2` | Merlot    	 	 | 35$		 |	Bev mo 						|
| `2`  | Coirander 	 	 | 1$		 |	Sprouts						|
| `3`  | Eggs		 	 	 | 6$		 |	Trader Joes's				|
| `3` | Milk		 	 	 | 3$		 |	Farmer's market				|

Now if the transformation filter-functions can be : 
1. Show me the record where the item is milk
2. Show me all items where the cost is higher than 5$
3. Show me all items where the name of the food ends with the letter `s`

Now if notice each of the above filter-functions, each of them can be applied to the each of the partitions(`1`,`2`,`3`) without depending on the other partition and the resultant dataset will be as expected. These are called as transformations. 

Following are a few examples to demonstrate that:

![transformation_example](https://github.com/JyotsnaP/Spark/blob/master/Images/transformation_example.png)



- Wide Transformation


map()
filter()
flatMap()

### Actions
count()
collect()
take(n)
top()
countByValue()
countByValue()
fold()
aggregate()
foreach()

### Spark

---
## Limitations of spark


