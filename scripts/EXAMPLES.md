# Spark 101 - Examples


## Spark

### Library structures: 

- from pyspark 
	- import SparkContext
- from pyspark.sql 
	- import SparkSession
	- import SQLContext
	- import Row
	- import Window

### Spark setup 

```
spark = SparkSession.builder.appName("Jyotsna Examples").enableHiveSupport().getOrCreate()
sc = SparkContext.getOrCreate()
sqlContext = SQLContext(Spark)
```

### Reading data:

##### Reading data from a CSV
- Reading data :
   - From a csv file into a DataFrame
   - From all the csv files in a directory
- Options when reading a csv


| OPTION 		 	 | FUNCTION 		 	 | 			  
|:------------------ |:------------------------------|
| `DELIMITER`      		 | Used to specify the delimiter of the csv file. By default it is `,(COMMA)`, but can be other characters like pipe, tab, space  	 
| `INFER SCHEMA`    	 | This is used if you want the schema to be inferred from the value in the columns. By default, it is set to `False`.       					 | 
| `HEADER` 	 | This is used to read the first line of the csv and use as column names. But default the value is set to `False` 			 | 
| `QUOTES`  | When you have a column with a delimiter that is used to split the columns, use `quotes` option to specify the quote character and delimiters inside the quotes are ignored. By default this is set to `"` 			 | 
| `NULL VALUES`  | This can be used in cases where you want to explicit consider a value in column as null. For ex, If you want a date column with a value of `1900-01-01` set null on the data frame | 
| `DATE FORMAT`  | This is used to set the format of the input DataType and TimestampTupe columns.  			 | 

### Examples: 

- #### This reads the `csv`/`json`/`parquet`/`text` file into the dataframe df
```
   df = spark.read.csv("/resources/example.csv")
   df = spark.read.json("/resources/json_file")
   df = spark.read.read("/resources/parquet_file_or_folder")
   df = spark.read.text("/resources/example.txt")
```
   
- #### Using the full source name and load you can do the same thing  
```
   df = spark.read.format("csv").load("/resources/example.csv")
```

-  #### When you run either of the above commands and then do a print schema you will not the schema to look something like this: 

```
df.printSchema()
root
	 |-- _c0: string (nullable = true)
	 |-- _c1: string (nullable = true)
	 |-- _c2: string (nullable = true)

# This shows that the first line of the CSV - which is typically the name of the columns have not been inferred, instead the columns are names _c1, _c2.
```

- #### To avoid the above, add the following option.
Using the option to infer schema
```
df = spark.read.option("header","True").csv("/resources/example.csv")
```

- #### The schema has been inferred below. 
```
df.printSchema()
	root
		|-- rank: integer (nullable = true)
		|-- student_name: string (nullable = true)
		|-- score: double (nullable = true)
		|-- year: integer (nullable = true)
```

- #### Reading mulitple CSV files
```
df = spark.read.csv("file1,file2,file3") 
# Notice that its one string with multiple paths that are comma separated.
```

- #### Reading a whole directory of csv files
```
spark.read.csv("Folder path")
```

- #### One way to explicitly mention the delimiter for a file
```
df = spark.read.options(delimiter = ',').csv("/resources/example.csv") # Notice that this is options, and not option
```

- #### The options command can be chained together as follows
```
df = spark.read.options(delimiter = ',',inferSchema='True',header='True').csv("/resources/example.csv") 

# Notice that this is options, and not option
```

- #### The "option" commands can be chained together as follows
```
df = spark.read.option("delimiter",",").option("inferSchema",True).option("header",True).csv("/resources/example.csv")
```

- #### CSV files can also be read with a predefined schema
```
schema = StructType().add("rank",IntegerType(),True). add("student_name",StringType(),True).add("score",DoubleType(),True).add("year",IntegerType())
df = spark.read.format("csv").option("header",True).schema(schema).load("/resources/example.csv")
```

#### NOTE : One thing to note is the order in which we call .option and .csv. First we mention .option(...) and then .csv(). The other way around will not work

---

### Creating data frames:

To be able to parallelize Collections in Driver program, Apache Spark provides SparkContext.parallelize() method. When a spark parallelize method is applied on a Collection, a new distributed dataset is created with specified number of paritions(if mentioned), and the elements of the collection are copied over to the RDD.

*NOTE*: parallelize() method(like many other concepts in Spark) is also `lazy`.  

### Manipulating data frames:

### Joins:

### Aggregates:

### Window functions:

### Writing data:

### Accumulators:

### Spark submit: