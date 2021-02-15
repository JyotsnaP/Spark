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
	- from pyspark.sql.tpyes
		- StringType
		- StructField
		- StructType
		- IntegerType
	- from pyspark.sql.functions
		- udf

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

- #### Direct method of creating a DF is
```
df = spark.read.csv("/resources/example.csv")
```

- #### Another way to create is by using `Row`
```
from pyspark.sql import Row

food1 = Row(foodName='Tiramisu',Cost=10,Type='Vegetarian')
food2 = Row(foodName='Aquafaba',Cost=5,Type='Vegan')

foood = [food1,food2]

df = spark.createDataFrame(food)

df.show()
```

- #### Another way to create a data frame from a textfile or just values:
For this however you will need the schema to be mentioned
```
rdd = spark.textFile(filePath)

schema = StructType([
        StructField("foodName", StringType(), True),
        StructField("Cost", IntegerType(), True),
        StructField("Type", StringType(), True)
    ])
   
df = spark.createDataFrame(rdd, schema)
```

Doing the same thing as above without the textfile
```
food_schema = StructType([StructField("FoodName",StringType(),True),StructField("Cost",IntegerType(),True)])

food = [("Tiramisu",10),("Doritos",5),("Merlot",15)]

rdd = spark.sparkContext.parallelize(food)

df = sqlContext.createDataFrame(rdd,food_schema)

df.printSchema()
```

### Manipulating data frames:
Dataframes abstract what RDDs do. RDDs are not represented tabular, relational database like representation - but dataframes do. 
Hence whatever you perform on a database table like - changing the type of a column, adding a new column and inferring its value, 
or removing a column, all of it can be done on dataframes as well. 

For the sake of the following examples, lets assume our df to look like when printed.
```
+---------------+----+------------+--------------+
|       FoodName|Cost|       Store|          Type|
+---------------+----+------------+--------------+
|       Tiramisu|  10|     Safeway|    Vegetarian|
|       Aquafaba|   1|     Safeway|         Vegan|
| Chicket Breast|   6|Trader Joe's|Non-Vegetarian|
+---------------+----+------------+--------------+
```


- #### Let's say you want to add a constant value of `Food` in a new column called `Item`
`lit` is used to add a new column in a Pyspark Dataframe by assigning a constant or literal value.
```
df = df.withColumn('item', F.lit('Food'))

O/P
---
+---------------+----+------------+--------------+----+
|       FoodName|Cost|       Store|          Type|item|
+---------------+----+------------+--------------+----+
|       Tiramisu|  10|     Safeway|    Vegetarian|Food|
|       Aquafaba|   1|     Safeway|         Vegan|Food|
| Chicket Breast|  	6|Trader Joe's|Non-Vegetarian|Food|
+---------------+----+------------+--------------+----+
```

- #### Let's say you want to add a column but keep it as null
```
df = df.withColumn('item', F.lit(None).cast(StringType()))

O/P
---
+---------------+----+------------+--------------+----+
|       FoodName|Cost|       Store|          Type|item|
+---------------+----+------------+--------------+----+
|       Tiramisu|  10|     Safeway|    Vegetarian|null|
|       Aquafaba|   1|     Safeway|         Vegan|null|
| Chicket Breast|  	6|Trader Joe's|Non-Vegetarian|null|
+---------------+----+------------+--------------+----+
```

- #### If you want to create a new column to categorize if the food is expensive or not
```
df = df. withColumn("range",F.when(F.col('Cost')<=5,'Inexpensive').when(F.col('Cost')>=10, 'Expensive').when((F.col('Cost')>5) & (F.col('Cost')<10),'Economic'))

O/P
---
+---------------+----+------------+--------------+----+-----------+
|       FoodName|Cost|       Store|          Type|item|      range|
+---------------+----+------------+--------------+----+-----------+
|       Tiramisu|  10|     Safeway|    Vegetarian|null|  Expensive|
|       Aquafaba|   1|     Safeway|         Vegan|null|Inexpensive|
| Chicket Breast|  6 |Trader Joe's|Non-Vegetarian|null|  Economic |
+---------------+----+------------+--------------+----+-----------+
```

- #### For modularizing your code, you can also a UDF(User Defined Function) to do the same
```
def range_categorize(val):
    if val <= 5:
       return 'Inexpensive'
    if val > 5 and val < 10:
       return 'Economic'
    if val >=10:
       return 'Expensive'


costElements = F.udf(lambda z: range_categorize(z), StringType())
spark.udf.register("costElements",costElements)
df = df.withColumn('range_categorize',costElements('Cost'))
df.show()


O/P
----
+---------------+----+------------+--------------+----+----------------+
|       FoodName|Cost|       Store|          Type|item|range_categorize|
+---------------+----+------------+--------------+----+----------------+
|       Tiramisu|  10|     Safeway|    Vegetarian|null|       Expensive|
|       Aquafaba|   1|     Safeway|         Vegan|null|     Inexpensive|
| Chicket Breast|   6|Trader Joe's|Non-Vegetarian|null|        Economic|
+---------------+----+------------+--------------+----+----------------+
```

**NOTE: One thing to observe is that df['Cost'] is a column object and we want its elements, hence we are using this simple 
lambda `costElements` to return all the elements of that column to the UDF. Also note that StringType() in that line refers
to the return type of the UDF**

- #### Apart from being able to create columns and manipulate the data in the new column, we can also rename columns.

Changing column name with withColumnRenamed feature
```
df = df.withColumnRenamed('range_categorize', 'range_categorization')
```

```
from pyspark.sql.functions import col
df = df.select(col("range_categorize").alias("range_categorization"), col("item").alias("food_item"))
```

- #### To be able to drop columns that are not required, we can do this. 
```
df = df.drop('range_categorize')
```

```
drop_columns = ['range_categorize', 'item']
df_new = df.select([col for col in df.columns if col not in drop_columns])
```

### Joins:

### Aggregates:

### Window functions:

### Writing data:

### Accumulators:

### Spark submit:

