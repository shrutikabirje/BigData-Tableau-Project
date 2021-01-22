import pyspark

from pyspark.sql.functions import *

from pyspark.context import SparkContext

from pyspark.sql import SQLContext

from pyspark.sql.session import SparkSession

sc = SparkContext()

sqlContext = SQLContext(sc)
spark = SparkSession.builder.master("local").appName("app name").config("spark.some.config.option", 'true').getOrCreate()

df = spark.read.parquet("s3://us-accidents-output/output/clean.parquet/")


df.createOrReplaceTempView("final_data")
sqlContext.sql("create table us_accidents select * from final_data")
