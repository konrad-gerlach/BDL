## PySpark Setup

### Prerequisites

You have already done the *Spark Setup* tutorial.

### Installation

First we add Spark to the Python path. Add the following line to the `~/.environment_variables` file and `source ~/.environment_variables` the changes.

~/.environment_variables:

```bash
export PYTHONPATH=$(ZIPS=("$SPARK_HOME"/python/lib/*.zip); IFS=:; echo "${ZIPS[*]}"):$PYTHONPATH
```

```bash
source ~/.environment_variables
```

```bash
#Copy the data to the cluster.
scp ./data_file pi@node01:~
```

Next we install Python, Pip and Venv with the following commands.

```bash
sudo apt update
sudo apt --yes upgrade
sudo apt --yes install python3-pip python3-venv
```

Now we create a new Venv, install PySpark and Jupiter and start a Jupyter Notebook.

```bash
python3 -m venv .venv/word_count
source .venv/word_count/bin/activate
pip install pyspark jupyter hdfs pandas sklearn seaborn
jupyter notebook --no-browser --port=8888 --ip=node01
```

Follow the instructions to access the Jupyter Notebook Web UI.

### Word Count Example

After opening the Jupyter web UI, create a new notebook and add the following cells.

```bash
# Import SparkSession
from pyspark.sql import SparkSession

# Create SparkSession
spark = SparkSession.builder \
    .master("spark://node01:7077") \
    .appName("WordCount") \
    .getOrCreate()
```

```bash
# Generate data  
with open("/home/pi/data_file.txt", "r") as f:
    lines = f.readlines()
    for _ in range(12):
        lines.extend(lines)
f.close()
```
```bash
# Write large file
with open("/home/pi/data_file_large.txt", "w") as f:
    f.writelines(lines)
f.close()
```

```bash
sc = spark.sparkContext

# Upload the data to HDFS
sc.parallelize(range(10)).saveAsTextFile("/home/pi/data_file_large.txt")

# Read the input file and Calculating words count
text_file = sc.textFile("/WordCount/input/data_file_large.txt")
counts = text_file.flatMap(lambda line: line.split(" ")) \
    .map(lambda word: (word, 1)) \
    .reduceByKey(lambda x, y: x + y)

# Collect and print the word count output
counts.collect()
```
