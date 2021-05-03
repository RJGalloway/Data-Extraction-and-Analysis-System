# Installing Spark

Installation of Apache Spark is very easy. In your home directory `~/`, get the latest [Apache Spark built for Hadoop 2.7](https://spark.apache.org/downloads.html), extract it and then move it to /opt:
```console
    cd ~/
    wget https://apache.osuosl.org/spark/spark-2.4.7/spark-2.4.7-bin-hadoop2.7.tgz
    tar xzvf spark-2.4.7-bin-hadoop2.7.tgz
    sudo mv ~/spark-2.4.7-bin-hadoop2.7 /opt/
    rm spark-2.4.7-bin-hadoop2.7.tgz
```

### Configuration

Next you want to update `spark-env.sh` which is inside the `conf` directory of Spark:
```console
    cd /opt/spark-2.4.7-bin-hadoop2.7/conf
    cp spark-env.sh.template spark-env.sh
    nano spark-env.sh
```

Uncomment `SPARK_MASTER_HOST`, `SPARK_MASTER_PORT / SPARK_MASTER_WEBUI_PORT`, and `SPARK_WORKER_MEMORY` and replace it with:

```console
    SPARK_MASTER_HOST=10.0.0.50  #to bind the master to a different IP address or hostname
    SPARK_MASTER_WEBUI_PORT=4040  #to use non-default ports for the master
    SPARK_WORKER_MEMORY=512m  #to set how much total memory workers have to give executors (e.g. 1000m, 2g)
```

Additionally we'll need to add the following to .bashrc:
```console
    nano ~/.bashrc
```

Add below the current `export PATH`:
```console
    # Spark
    export PATH=$PATH:/opt/spark-2.4.7-bin-hadoop2.7/bin
```

Changes in the .bashrc are not applied when you save this file. Refresh the environment variables by typing:
```console
    source ~/.bashrc
```

### Boot Spark

From **NameNode** only, run the following commands:
```console
    cd /opt/spark-2.4.7-bin-hadoop2.7/bin
    spark-class org.apache.spark.deploy.master.Master --port 7077 --webui-port 4040
```

You'll then be able to access the UI by going to:

- [Spark Master - Web UI](http://192.168.1.58:4040/)


## Test Spark

### Spark in Standalone Mode

[Source](https://gist.github.com/datalove/5dbb69936d7284601f3e)

#### Test Spark

Navigate to the following directory:
```console
    cd /opt/spark-2.4.7-bin-hadoop2.7/bin
```

The following command calculates Pi to 10 decimals:
```console
    run-example SparkPi 10
```

#### Test Scala Shell

Navigate to the following directory:
```console
    cd /opt/spark-2.4.7-bin-hadoop2.7/bin
```

Run the following commands:
```console
    spark-shell --master local[4]
    scala> sc.textFile("README.md").count
    To see what spark is doing, go to [http://raspi08.home:4040/]
    ctrl+D quits the shell
```

#### Test Python Shell

Navigate to the following directory:
```console
    cd /opt/spark-2.4.7-bin-hadoop2.7/bin

    pyspark --master local[4]
    >>> sc.textFile("README.md").count()
    ctrl+D quites the shell
```
