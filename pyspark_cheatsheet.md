# PySpark Cheat Sheet

A practical reference for PySpark operations commonly used in data engineering and ETL workflows.

---

## Session & Configuration

### Create a Spark session

```python
spark = SparkSession.builder.appName("my_app").getOrCreate()
```

### Set a Spark config option

```python
spark.conf.set("spark.sql.shuffle.partitions", "200")
```

### Stop the Spark session

```python
spark.stop()
```

---

## Reading Data

### Read a CSV file

```python
df = spark.read.csv("path/to/file.csv", header=True, inferSchema=True)
```

### Read a JSON file

```python
df = spark.read.json("path/to/file.json")
```

### Read a Parquet file

```python
df = spark.read.parquet("path/to/file.parquet")
```

### Read a Delta table

```python
df = spark.read.format("delta").load("path/to/delta_table")
```

### Read from a JDBC source (e.g. PostgreSQL)

```python
df = spark.read.jdbc(url="jdbc:postgresql://host:5432/db", table="schema.table", properties={"user": "user", "password": "pass"})
```

---

## Inspecting Data

### Show the first N rows

```python
df.show(10)
```

### Print the schema

```python
df.printSchema()
```

### Get column names

```python
df.columns
```

### Get number of rows and columns

```python
df.count(), len(df.columns)
```

### Show basic statistics

```python
df.describe().show()
```

---

## Selecting & Filtering

### Select columns

```python
df.select("col1", "col2")
```

### Filter rows by condition

```python
df.filter(df["age"] > 30)
```

### Filter with multiple conditions

```python
df.filter((df["age"] > 30) & (df["country"] == "Finland"))
```

### Filter using SQL-style string

```python
df.filter("age > 30 AND country = 'Finland'")
```

### Select with an expression

```python
from pyspark.sql.functions import col, expr
df.select(col("salary") * 1.1)
df.select(expr("salary * 1.1 as adjusted_salary"))
```

---

## Transforming Columns

### Rename a column

```python
df = df.withColumnRenamed("old_name", "new_name")
```

### Add a new column

```python
df = df.withColumn("new_col", df["salary"] * 1.1)
```

### Drop a column

```python
df = df.drop("col_name")
```

### Cast a column to a different type

```python
df = df.withColumn("age", df["age"].cast("integer"))
```

### Apply a built-in function to a column

```python
from pyspark.sql.functions import upper, trim, to_date
df = df.withColumn("name", upper(trim(df["name"])))
df = df.withColumn("date", to_date(df["date_str"], "yyyy-MM-dd"))
```

### Replace values in a column

```python
df = df.replace("N/A", None, subset=["col_name"])
```

### Create a conditional column

```python
from pyspark.sql.functions import when
df = df.withColumn("level", when(df["score"] >= 90, "high").when(df["score"] >= 60, "mid").otherwise("low"))
```

---

## Cleaning Data

### Drop rows with null values

```python
df = df.dropna()
```

### Drop rows where specific columns are null

```python
df = df.dropna(subset=["col1", "col2"])
```

### Fill null values

```python
df = df.fillna({"salary": 0, "department": "unknown"})
```

### Drop duplicate rows

```python
df = df.dropDuplicates()
```

### Drop duplicates based on specific columns

```python
df = df.dropDuplicates(["email", "employee_id"])
```

---

## Aggregation & Grouping

### Group by and aggregate

```python
df.groupBy("department").agg({"salary": "avg", "id": "count"}).show()
```

### Group by with named aggregations

```python
from pyspark.sql.functions import avg, count, sum, max, min
df.groupBy("department").agg(
    avg("salary").alias("avg_salary"),
    count("id").alias("headcount")
).show()
```

### Count distinct values in a column

```python
from pyspark.sql.functions import countDistinct
df.select(countDistinct("customer_id")).show()
```

### Get distinct rows

```python
df.select("col_name").distinct()
```

---

## Sorting

### Sort by a column ascending

```python
df = df.orderBy("col_name")
```

### Sort by a column descending

```python
df = df.orderBy("col_name", ascending=False)
```

### Sort by multiple columns

```python
df = df.orderBy(["country", "salary"], ascending=[True, False])
```

---

## Joining & Combining

### Join two DataFrames

```python
df1.join(df2, on="id", how="inner")
```

### Join on multiple keys

```python
df1.join(df2, on=["id", "date"], how="left")
```

### Union two DataFrames (same schema)

```python
df1.union(df2)
```

### Union and remove duplicates

```python
df1.unionByName(df2).distinct()
```

---

## SQL Interface

### Register a temporary view

```python
df.createOrReplaceTempView("my_table")
```

### Run a SQL query

```python
result = spark.sql("SELECT department, AVG(salary) FROM my_table GROUP BY department")
```

---

## Window Functions

### Rank rows within a group

```python
from pyspark.sql.functions import rank
from pyspark.sql.window import Window

window = Window.partitionBy("department").orderBy("salary")
df = df.withColumn("rank", rank().over(window))
```

### Running total within a group

```python
from pyspark.sql.functions import sum as spark_sum

window = Window.partitionBy("department").orderBy("date").rowsBetween(Window.unboundedPreceding, 0)
df = df.withColumn("running_total", spark_sum("sales").over(window))
```

---

## Performance & Partitioning

### Cache a DataFrame in memory

```python
df.cache()
```

### Unpersist a cached DataFrame

```python
df.unpersist()
```

### Repartition (increases or decreases partitions, causes shuffle)

```python
df = df.repartition(8)
```

### Coalesce (reduces partitions, no full shuffle)

```python
df = df.coalesce(1)
```

### Check the number of partitions

```python
df.rdd.getNumPartitions()
```

---

## Writing Data

### Write to CSV

```python
df.write.csv("path/to/output", header=True, mode="overwrite")
```

### Write to Parquet

```python
df.write.parquet("path/to/output", mode="overwrite")
```

### Write to Delta

```python
df.write.format("delta").mode("overwrite").save("path/to/delta_table")
```

### Write partitioned by a column

```python
df.write.partitionBy("country").parquet("path/to/output")
```

### Write to a JDBC target

```python
df.write.jdbc(url="jdbc:postgresql://host:5432/db", table="schema.table", mode="append", properties={"user": "user", "password": "pass"})
```

---

## Interoperability

### Convert to Pandas DataFrame

```python
pdf = df.toPandas()
```

### Create a Spark DataFrame from Pandas

```python
df = spark.createDataFrame(pdf)
```

### Create a Spark DataFrame manually

```python
from pyspark.sql import Row
data = [Row(name="Ali", age=30), Row(name="Sara", age=25)]
df = spark.createDataFrame(data)
```

---

## Schema

### Define a schema explicitly

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

schema = StructType([
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True)
])
df = spark.read.csv("path/to/file.csv", schema=schema, header=True)
```

---

## ETL Patterns

### Count value frequency in a column

```python
df.groupBy("status").count().orderBy("count", ascending=False).show()
```

### Top N rows per group

```python
from pyspark.sql.functions import row_number
from pyspark.sql.window import Window

window = Window.partitionBy("department").orderBy(col("salary").desc())
df.withColumn("rn", row_number().over(window)).filter(col("rn") <= 3).drop("rn")
```

### Deduplicate keeping the latest record

```python
from pyspark.sql.functions import row_number
from pyspark.sql.window import Window

window = Window.partitionBy("customer_id").orderBy(col("updated_at").desc())
df = df.withColumn("rn", row_number().over(window)).filter(col("rn") == 1).drop("rn")
```

### Null audit — count nulls across all columns

```python
from pyspark.sql.functions import col, sum as spark_sum

df.select([spark_sum(col(c).isNull().cast("int")).alias(c) for c in df.columns]).show()
```

### Incremental load — filter rows newer than a watermark

```python
df = df.filter(col("updated_at") > last_watermark)
new_watermark = df.agg({"updated_at": "max"}).collect()[0][0]
```

### Detect changed rows between two snapshots (SCD check)

```python
changed = df_new.join(df_old, on="id", how="left") \
    .filter(df_new["value"] != df_old["value"])
```

### Pivot — turn row values into columns

```python
df.groupBy("employee_id").pivot("quarter").agg(sum("revenue")).show()
```

### Unpivot — turn columns into rows

```python
df.selectExpr("id", "stack(3, 'q1', q1, 'q2', q2, 'q3', q3) as (quarter, revenue)")
```

### Enrich a fact table by joining a lookup/dimension table

```python
df_enriched = df_orders.join(df_customers, on="customer_id", how="left") \
    .select("order_id", "amount", "customer_name", "country")
```

### Standardize and clean a string column in one step

```python
from pyspark.sql.functions import lower, trim, regexp_replace

df = df.withColumn("name", lower(trim(regexp_replace(col("name"), "[^a-zA-Z ]", ""))))
```

### Flag bad rows instead of dropping them

```python
df = df.withColumn("is_invalid", when(col("email").isNull() | col("age") < 0, True).otherwise(False))
df_clean = df.filter(col("is_invalid") == False)
df_quarantine = df.filter(col("is_invalid") == True)
```

### Aggregate with multiple metrics in one pass

```python
from pyspark.sql.functions import avg, count, sum, max, min, stddev

df.groupBy("region").agg(
    count("order_id").alias("total_orders"),
    sum("revenue").alias("total_revenue"),
    avg("revenue").alias("avg_revenue"),
    max("revenue").alias("max_revenue"),
    stddev("revenue").alias("stddev_revenue")
).show()
```

### Broadcast join — optimized join when one table is small

```python
from pyspark.sql.functions import broadcast

df_result = df_large.join(broadcast(df_small), on="id", how="inner")
```

### Write bad records to a quarantine path, good records to the target

```python
df_clean.write.parquet("path/to/output/clean", mode="append")
df_quarantine.write.parquet("path/to/output/quarantine", mode="append")
```

### Read multiple files from a folder at once

```python
df = spark.read.parquet("path/to/folder/*.parquet")
```

### Add a surrogate key (unique row ID)

```python
from pyspark.sql.functions import monotonically_increasing_id

df = df.withColumn("surrogate_key", monotonically_increasing_id())
```

---

## Spark ML

---

### Shared Pipeline Steps

These steps are the same regardless of whether you are doing regression, classification, or clustering.

#### 1. Import libraries

```python
from pyspark.sql import SparkSession
from pyspark.ml.feature import VectorAssembler, StringIndexer, StandardScaler
from pyspark.ml import Pipeline
```

#### 2. Create a Spark session

```python
spark = SparkSession.builder.appName("ml_pipeline").getOrCreate()
```

#### 3. Load the dataset

```python
df = spark.read.csv("data.csv", header=True, inferSchema=True)
```

#### 4. Encode categorical target column (classification only)

If your label column is a string, convert it to a numeric index first.

```python
indexer = StringIndexer(inputCol="label", outputCol="label_index")
df = indexer.fit(df).transform(df)
```

#### 5. Assemble feature columns into a single vector

```python
assembler = VectorAssembler(
    inputCols=["feature1", "feature2", "feature3"],
    outputCol="features"
)
df = assembler.transform(df)
```

#### 6. (Optional) Scale features

Recommended for models sensitive to feature magnitude (e.g. linear models, SVM).

```python
scaler = StandardScaler(inputCol="features", outputCol="scaled_features")
df = scaler.fit(df).transform(df)
```

#### 7. Split into training and test sets

Not used in clustering — skip this step there.

```python
train_df, test_df = df.randomSplit([0.8, 0.2], seed=42)
```

---

### Regression

Predicts a continuous numeric value.

#### Available models

| Model                  | Import                                                    |
| ---------------------- | --------------------------------------------------------- |
| Linear Regression      | `from pyspark.ml.regression import LinearRegression`      |
| Decision Tree          | `from pyspark.ml.regression import DecisionTreeRegressor` |
| Random Forest          | `from pyspark.ml.regression import RandomForestRegressor` |
| Gradient Boosted Trees | `from pyspark.ml.regression import GBTRegressor`          |

#### Train a model

```python
from pyspark.ml.regression import RandomForestRegressor

model = RandomForestRegressor(featuresCol="features", labelCol="price", numTrees=100)
trained_model = model.fit(train_df)
```

#### Make predictions

```python
predictions = trained_model.transform(test_df)
predictions.select("price", "prediction").show(10)
```

#### Evaluate

```python
from pyspark.ml.evaluation import RegressionEvaluator

evaluator = RegressionEvaluator(labelCol="price", predictionCol="prediction")

rmse = evaluator.setMetricName("rmse").evaluate(predictions)
mae  = evaluator.setMetricName("mae").evaluate(predictions)
r2   = evaluator.setMetricName("r2").evaluate(predictions)

print(f"RMSE: {rmse:.4f} | MAE: {mae:.4f} | R²: {r2:.4f}")
```

---

### Classification

Predicts a discrete class label.

#### Available models

| Model                  | Import                                                         |
| ---------------------- | -------------------------------------------------------------- |
| Logistic Regression    | `from pyspark.ml.classification import LogisticRegression`     |
| Decision Tree          | `from pyspark.ml.classification import DecisionTreeClassifier` |
| Random Forest          | `from pyspark.ml.classification import RandomForestClassifier` |
| Gradient Boosted Trees | `from pyspark.ml.classification import GBTClassifier`          |
| Naive Bayes            | `from pyspark.ml.classification import NaiveBayes`             |
| Linear SVM             | `from pyspark.ml.classification import LinearSVC`              |

#### Train a model

```python
from pyspark.ml.classification import RandomForestClassifier

model = RandomForestClassifier(featuresCol="features", labelCol="label_index", numTrees=100)
trained_model = model.fit(train_df)
```

#### Make predictions

```python
predictions = trained_model.transform(test_df)
predictions.select("label_index", "prediction", "probability").show(10)
```

#### Evaluate

```python
from pyspark.ml.evaluation import MulticlassClassificationEvaluator, BinaryClassificationEvaluator

evaluator = MulticlassClassificationEvaluator(labelCol="label_index", predictionCol="prediction")

accuracy  = evaluator.setMetricName("accuracy").evaluate(predictions)
f1        = evaluator.setMetricName("f1").evaluate(predictions)
precision = evaluator.setMetricName("weightedPrecision").evaluate(predictions)
recall    = evaluator.setMetricName("weightedRecall").evaluate(predictions)

# AUC — for binary classification only
auc_evaluator = BinaryClassificationEvaluator(labelCol="label_index", metricName="areaUnderROC")
auc = auc_evaluator.evaluate(predictions)

print(f"Accuracy: {accuracy:.4f} | F1: {f1:.4f} | Precision: {precision:.4f} | Recall: {recall:.4f} | AUC: {auc:.4f}")
```

---

### Clustering

Groups data into clusters without a target label. No train/test split — the full dataset is used.

#### Available models

| Model                  | Import                                              |
| ---------------------- | --------------------------------------------------- |
| K-Means                | `from pyspark.ml.clustering import KMeans`          |
| Bisecting K-Means      | `from pyspark.ml.clustering import BisectingKMeans` |
| Gaussian Mixture Model | `from pyspark.ml.clustering import GaussianMixture` |

#### Train a model

```python
from pyspark.ml.clustering import KMeans

model = KMeans(featuresCol="features", k=3, seed=42)
trained_model = model.fit(df)
```

#### Make predictions

```python
predictions = trained_model.transform(df)
predictions.select("features", "prediction").show(10)
```

#### Evaluate

```python
from pyspark.ml.evaluation import ClusteringEvaluator

evaluator = ClusteringEvaluator(featuresCol="features", metricName="silhouette")
silhouette = evaluator.evaluate(predictions)

print(f"Silhouette Score: {silhouette:.4f}")
# Score ranges from -1 to 1. Closer to 1 means well-separated clusters.
```

#### Find the best K — elbow method

```python
costs = []
for k in range(2, 10):
    model = KMeans(featuresCol="features", k=k, seed=42)
    trained = model.fit(df)
    costs.append((k, trained.summary.trainingCost))

for k, cost in costs:
    print(f"k={k}  cost={cost:.2f}")
```

---

### Hyperparameter Tuning

#### Grid search with cross-validation

```python
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator

param_grid = ParamGridBuilder() \
    .addGrid(model.numTrees, [50, 100, 200]) \
    .addGrid(model.maxDepth, [3, 5, 10]) \
    .build()

cv = CrossValidator(
    estimator=model,
    estimatorParamMaps=param_grid,
    evaluator=evaluator,
    numFolds=3
)

cv_model = cv.fit(train_df)
best_model = cv_model.bestModel
```

---

### Using a Pipeline

Chains all steps into one clean object — recommended for production.

```python
from pyspark.ml import Pipeline

assembler = VectorAssembler(inputCols=["f1", "f2", "f3"], outputCol="features")
scaler = StandardScaler(inputCol="features", outputCol="scaled_features")
model = RandomForestClassifier(featuresCol="scaled_features", labelCol="label_index")

pipeline = Pipeline(stages=[assembler, scaler, model])
pipeline_model = pipeline.fit(train_df)
predictions = pipeline_model.transform(test_df)
```

---

### Saving & Loading Models

```python
# Save
trained_model.save("models/my_model")
pipeline_model.save("models/my_pipeline")

# Load
from pyspark.ml.classification import RandomForestClassificationModel
from pyspark.ml import PipelineModel

loaded_model = RandomForestClassificationModel.load("models/my_model")
loaded_pipeline = PipelineModel.load("models/my_pipeline")
```

---

_Last updated: 2026_
