
# Spark-RevoScaleR-Examples
This is a collection of R scripts I used in different educational sessions on how to use R with Spark.

The test dataset is New York City taxi data, on a Azure HDInsight cluster, from the edge node with R Server installed, we can download the file with wget command:

wget https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2016-01.csv

Once the file downloaded (1.5GB) you can put it into HDFS:
hadoop fs -mkdir /tmp/nyc
hadoop fs -put yellow_tripdata_2016-01.csv /tmp/nyc

The HDI cluster (v 3.6) has R 9.1 installed and has 4 D14-V2 work nodes.

We show how to read the csv file into a dataframe and then call the Generalized Linear Regression (rxGlm) to create a linear model on the dataset.

We show the rxComputeContext setting for local, localpar, and Spark. For the last we also try a few different parameters to see how they impact the resource usage.

rxSetComputeContext("local")
Elapsed computation time: 186.197 secs.
Comments: This setting uses all cores on the edge node.

rxSetComputeContext("localpar")
Elapsed computation time: 185.533 secs.
Comments: This setting is to use all cores in all edge nodes. Since we only have one, so no effect.

myHadoopCluster <- rxSparkConnect(reset=TRUE)
Time: user system elapsed 1.606 0.784 71.230
Comments: It seems the default setting is the best one for our problem.

myHadoopCluster <- rxSparkConnect(reset=TRUE,numExecutors=32)
Time: user system elapsed 1.616 0.748 69.230
Comments: For parameters the int cannot put into quotation makrs.

myHadoopCluster <- rxSparkConnect(reset=TRUE,numExecutors=”32”)
Time: user system elapsed 4.624 0.304 191.137
Comments: Only you put the int in quotation mark, it falls back to local one.

myHadoopCluster <- rxSparkConnect(reset=TRUE,executorCores=6)
Time: user system elapsed 1.676 1.024 69.342

myHadoopCluster <- rxSparkConnect(reset=TRUE,numExecutors=4,executorCores=6,executorMem="2g")
Time: user system elapsed 1.820 1.124 71.579
Comments: We do see the Memory in Use from Yarn monitor is 4x6x2=48GB.

myHadoopCluster <- rxSparkConnect(reset=TRUE,numExecutors=6,executorCores=6,executorMem="2g")
Time: user system elapsed 1.664 1.032 71.356
Comments: We do see the Memory in Use from Yarn monitor is 6x6x2=72GB.

myHadoopCluster <- rxSparkConnect(reset=TRUE,numExecutors=12,executorCores=6,executorMem="2g")
Time: user system elapsed 1.832 1.184 83.746
Comments: The required 12x6x2=144GB is beyond what we have.

Finally, we show examples on how to retrieve data from Hive (table hivesampletable) and parquet files (in /share/claimsParquet) and utlize the data to build models.

