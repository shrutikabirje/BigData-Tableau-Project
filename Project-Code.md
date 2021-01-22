# LINKS
=======
https://smoosavi.org/datasets/us_accidents

https://sparkbyexamples.com/pyspark/pyspark-split-dataframe-column-into-multiple-columns/

https://blog.clairvoyantsoft.com/big-data-file-formats-3fb659903271

## not required this command## df = spark.read.csv("ProjectData/US_Accidents_June20.csv")

# FEATURES
==========

ID: This is a unique identifier of the accident record.

Source: Indicates source of the accident report (i.e. the API which reported the accident.).

Severity: Shows the severity of the accident, a number between 1 and 4, where 1 indicates the least impact on traffic (i.e., short delay as a result of the accident) and 
4 indicates a significant impact on traffic (i.e., long delay).

Start_Time: Shows start time of the accident in local time zone.

End_Time: Shows end time of the accident in local time zone. 
End time here refers to when the impact of accident on traffic flow was dismissed.

Start_Lat: Shows latitude in GPS coordinate of the start point.

Start_Lng: Shows longitude in GPS coordinate of the start point.

Distance(mi): The length of the road extent affected by the accident.

Description: Shows natural language description of the accident.

Street: Shows the street name in address field.

Side: Shows the relative side of the street (Right/Left) in address field.

City: Shows the city in address field.

County: Shows the county in address field.

State: Shows the state in address field.

Zipcode: Shows the zipcode in address field.

Country: Shows the country in address field.

Timezone: Shows timezone based on the location of the accident (eastern, central, etc.).

Airport_Code: Denotes an airport-based weather station which is the closest one to location of the accident.

Weather_Timestamp: Shows the time-stamp of weather observation record (in local time).

Temperature(F): Shows the temperature (in Fahrenheit).

Wind_Chill(F): Shows the wind chill (in Fahrenheit)

Visibility(mi): Shows visibility (in miles).

Wind_Direction: Shows wind direction.

Wind_Speed(mph): Shows wind speed (in miles per hour).

Weather_Condition: Shows the weather condition (rain, snow, thunderstorm, fog, etc.)

Bump: A POI annotation which indicates presence of speed bump or hump in a nearby location.

Crossing: A POI annotation which indicates presence of crossing in a nearby location.

Junction: A POI annotation which indicates presence of junction in a nearby location.

Railway: A POI annotation which indicates presence of railway in a nearby location.

Station: A POI annotation which indicates presence of station in a nearby location.

Traffic_Calming: A POI annotation which indicates presence of traffic_calming in a nearby location.

Traffic_Signal: A POI annotation which indicates presence of traffic_signal in a nearby location.

Sunrise_Sunset: Shows the period of day (i.e. day or night) based on sunrise/sunset.

# STEPS
=======
STEP 1:
s3 to hdfs data ingestion ::

hdfs dfs -cp s3n://us-accident/US_Accidents_June20.csv ProjectData/

#STEP 2:
from pyspark.sql import SQLContext

df = sqlContext.read.format('com.databricks.spark.csv').options(header='true',inferschema='true').load("ProjectData/US_Accidents_June20.csv")

df.count()

df.printSchema()


# Rename a column name
======================

df=df.withColumnRenamed("Distance(mi)","Distance")

df=df.withColumnRenamed("Temperature(F)","Temperature")

df=df.withColumnRenamed("Wind_Chill(F)","Wind_Chill")

df=df.withColumnRenamed("Humidity(%)","Humidity")

df=df.withColumnRenamed("Pressure(in)","Pressure")

df=df.withColumnRenamed("Visibility(mi)","Visibility")

df=df.withColumnRenamed("Wind_Speed(mph)","Wind_Speed")

df=df.withColumnRenamed("Precipitation(in)","Precipitation")


# Drop column which is not required
===================================

df = df.drop("TMC","Source","End_Lat","End_Lng","Number","County","Precipitation")

df.printSchema()

df.describe().show()


#Now Replace NaN values with mean on some features
===================================================

from pyspark.sql.functions import mean

mean_press=clean_df.select(mean(clean_df.Pressure)).collect()

clean_df.na.fill(mean_press,subset=['Pressure']).show()



# Store this dataframe into Hive Table
======================================

df.createOrReplaceTempView("us_accident")

sqlContext.sql("create table US_accident_Data select * from us_accident")

clean_df.describe().show()

df.select("Zipcode")
df.select("Zipcode").show(20)

from pyspark.sql.functions import split,regexp_replace

df=df.withColumn("zipcode",split(df["Zipcode"],'-').getItem(0))

df.select("Zipcode").show(20)


df = df.withColumn('Date',split_col.getItem(0))
df.select('Date).show(20)

df = df.withColumn('Time',split_col.getItem(1))
df.select('Time).show(20)


# Write dataframe in Parquet directly to S3
===========================================

df.write.parquet('/user/hadoop/us_accidents')
"s3://us-accidents-output/output/clean.parquet"
