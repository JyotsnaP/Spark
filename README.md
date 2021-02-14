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
|:-------------------------- |:---------------------------------------------------|
| `SPEED`      			 	 | Faster than single machine 	 | 100X faster than MR   |   			 	 
| `WRITTEN IN`    	 | Java      						 	  | Scala      			 |	 
| `EASE OF USE` 	 | Complex and lengthy 						 	  | Simple and crisp    |  			 	 
| `DATA PROCESSING` 	 	 | Batch processing 							 	 | Batch/Real time/iterative/graph |     			 	 
| `CACHING` 	 	 | Does not support caching, writes data to disk back and forth creating an I/O bottleneck    	  | Caches the data in-memory, enhancing the performance |     			 	 

## Spark jargons
-  ** Distributed computing**
-- Partioned data
-- Fault Tolerance
-- Lazy evaluation
-  **Spark as a process framework**
-- RDDS
-- Dataframes
-- Datasets
-- Transformations
-- Actions
-- Spark Job
---

### Partitioned Data
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
|:-------------------------- |:---------------------------------------------------|
| `STORAGE`      			 	 | Not stored in columnar format. They are stored as list of rows 	 | They are stored in columnar format  |  
| `SCHEMA`      			 	 | No schema 	 | Has all the features of an RDD but also has a schema. This is the my chice of data structure |while coding in Pyspark     			 	 


	![DF_has_schema](https://github.com/JyotsnaP/Spark/images/DF_has_schema)
--

	![RDD_does_not_schema](https://github.com/JyotsnaP/Spark/images/RDD_does_not_schema)
--

	![RDD_is_a_list_of_rows](https://github.com/JyotsnaP/Spark/images/RDD_is_a_list_of_rows)


---
## Limitations of spark


