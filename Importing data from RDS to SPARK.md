# Access data from rds to pyspark 

## Loading Data

Step1:Download the jar_files.zip 

step2:unzip the file

step3:sudo cp mysql-connector-java-5.1.49.jar /lib/spark/jars/

step4:go to pyspark

step5:establish the connection

from  pyspark.sql import SparkSession

from pyspark.sql import SQLContext

spark = SparkSession.builder.config(conf=SparkConf()).getOrCreate()

sqlContext = SQLContext(spark)

hostname='shnakidb.cqb3dula51ug.us-east-1.rds.amazonaws.com'

jdbcPort=3306

dbname='test'

username='admin'

password='admin123'

jdbc_url = "jdbc:mysql://{0}:{1}/{2}".format(hostname, jdbcPort, dbname)
connectionProperties = {
   "user" : username,
   "password" : password
}

step6:read the csv file 

df=spark.read.jdbc(url=jdbc_url,table='project',properties= connectionProperties)

step7:show the records

df.show()

