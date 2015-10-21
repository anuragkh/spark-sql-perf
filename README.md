# Spark SQL Performance Tests

[![Build Status](https://travis-ci.org/databricks/spark-sql-perf.svg)](https://travis-ci.org/databricks/spark-sql-perf)

This is a performance testing framework for [Spark SQL](https://spark.apache.org/sql/) in [Apache Spark](https://spark.apache.org/) 1.4+.

**Note: This README is still under development. Please also check our source code for more information.**

## How to use it
The rest of document will use TPC-DS benchmark as an example. We will add contents to explain how to use other benchmarks add the support of a new benchmark dataset in future.

### Setup a benchmark

To setup TPC-DS, perform the following steps:

```
git clone https://github.com/grahn/tpcds-kit.git
cd tpcds-kit/tools
sudo yum -y install gcc make flex bison byacc
make -f Makefile.suite
/root/spark-ec2/copy-dir --delete /root/tpcds-kit
```

Before running any query, a dataset needs to be setup by creating a `Benchmark` object.

```
import com.databricks.spark.sql.perf.tpcds.Tables
// Tables in TPC-DS benchmark used by experiments.
// dsdgenDir is the location of dsdgen tool installed in your machines.
val tables = new Tables(sqlContext=sqlContext, dsdgenDir="/root/tpcds-kit", scaleFactor=1)
// Generate data.
tables.genData(location="/tpcds", format="parquet", overwrite=false, partitionTables=false, useDoubleForDecimal=true, clusterByPartitionColumns=false, filterOutNullPartitionValues=false)
// Create metastore tables in a specified database for your data.
// Once tables are created, the current database will be switched to the specified database.
tables.createExternalTables(location="/tpcds", format="parquet", databaseName="tpcds", overwrite=true)
// Or, if you want to create temporary tables
tables.createTemporaryTables(location="/tpcds", format="parquet")
// Setup TPC-DS experiment
import com.databricks.spark.sql.perf.tpcds.TPCDS
val tpcds = new TPCDS (sqlContext=sqlContext)
```

### Run benchmarking queries
After setup, users can use `runExperiment` function to run benchmarking queries and record query execution time. Taking TPC-DS as an example, you can start an experiment by using

```
val experiment = tpcds.runExperiment(queriesToRun = tpcds.interactiveQueries)
```

For every experiment run (i.e.\ every call of `runExperiment`), Spark SQL Perf will use the timestamp of the start time to identify this experiment. Performance results will be stored in the sub-dir named by the timestamp in the given `resultsLocation` (for example `results/1429213883272`). The performance results are stored in the JSON format.

### Retrieve results
While the experiment is running you can use `experiment.html` to list the status.  Once the experiment is complete, the results will be saved to the table sqlPerformance in json.

```
// Get all experiments results.
tpcds.createResultsTable()
sqlContext.sql("sqlPerformance")
// Get the result of a particular run by specifying the timestamp of that run.
sqlContext.sql("sqlPerformance").filter("timestamp = 1429132621024")
```