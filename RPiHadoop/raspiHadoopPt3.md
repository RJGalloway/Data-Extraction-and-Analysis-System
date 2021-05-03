# Raspberry Pi Hadoop Cluster (Part 3) - Building a Raspberry Pi Hadoop Cluster (DataNodes)

_Revision Date: 4/22/2021_

[Source](https://web.archive.org/web/20170215170838/http://www.becausewecangeek.com/building-a-raspberry-pi-hadoop-cluster-part-2/)


## Preparing DataNodes

### User Permissions

Every node needs to have a Hadoop user, so we can reuse a bunch of commands from the previous part.
```console
    sudo addgroup hadoop
    sudo adduser --ingroup hadoop hduser
```

> Press ENTER for all user information to select default; press Y to save
```console
    sudo adduser hduser sudo
```

Everything Hadoop will be happening via the `hduser`. Let's change to this user.
```console
    su hduser
```

Next, we'll need to make sure that each of our Raspberry Pi's can be accessed via an address. Update `/etc/hosts`:
```console
    sudo nano /etc/hosts
```

What you fill in here is very dependent on your network setup. Make sure that the IP addresses that are assigned to your Hadoop nodes are static (or at least very unlikely to change). A little example of which lines you can add to your /etc/hosts file.
```console
    #/etc/hosts
    127.0.0.1         localhost
    ::1               localhost ip6-localhost ip6-loopback
    ff02::1           ip6-allnodes
    ff02::2           ip6-allrouters

    127.0.0.1         raspberrypi-namenode ### Type: `hostname` <Your Host Name> ###

    10.0.0.50         RaspberryPiHadoopNameNode
    10.0.0.51         RaspberryPiHadoopDataNode1
    10.0.0.52         RaspberryPiHadoopDataNode2
```

### Installing Java8

Execute the following command to install Java8:
```console
    sudo apt install openjdk-8-jre-headless openjdk-8-jdk-headless
```

**!! STOP HERE AND REPEAT STEPS FOR EACH DATANODE !!**

---

## SSH & Hadoop Setup on DataNode

### SSH Key

Continue the steps below on the **NameNode**.

Now copy the key to each of the data nodes so that they can be SSH into without a password:
```console
    ssh-copy-id hduser@RaspberryPiHadoopDataNode1
    ssh-copy-id hduser@RaspberryPiHadoopDataNode2
```

Now check to see if you can SSH to the DataNodes that you copied the key to:
```console
    ssh 'hduser@RaspberryPiHadoopDataNode1'
    exit

    ssh 'hduser@RaspberryPiHadoopDataNode2'
    exit
```

### Hadoop Installation to DataNodes

Now we want to copy our Hadoop installation to the other nodes. On **NameNode**:
```console
    cd ~/
    sudo apt-get install zip
    zip -r hadoop-2.9.2-with-armhf-drivers.zip /opt/hadoop-2.9.2/
```

Copy NameNode's Bash file to each data node:
```console
    scp ~/.bashrc hduser@RaspberryPiHadoopDataNode1:~/.bashrc
    ...
    scp ~/.bashrc hduser@RaspberryPiHadoopDataNode2:~/.bashrc
```

The archive is about 210 megabytes of Hadoop data. Thanks to our passwordless-SSH setup we can easily transfer to this the other nodes.

Secure copy zip to DataNode and expand it with correct privileges then delete the zip:

> **Make sure to update the hostname `RaspberryPiHadoopDataNode1` for each host.**
```console
    scp ~/hadoop-2.9.2-with-armhf-drivers.zip hduser@RaspberryPiHadoopDataNode1:~/
    ssh hduser@RaspberryPiHadoopDataNode1
    sudo unzip hadoop-2.9.2-with-armhf-drivers.zip -d /
    sudo chown -R hduser:hadoop /opt/hadoop-2.9.2/
    rm hadoop-2.9.2-with-armhf-drivers.zip
    sudo mkdir -p /hdfs/tmp
    sudo chown hduser:hadoop /hdfs/tmp
    chmod 750 /hdfs/tmp

    exit

```
**!! STOP HERE AND REPEAT STEPS FOR EACH DATANODE !!**
---


### Boot The Cluster

From **NameNode** only, run the following commands:
```console
    cd $HADOOP_CONF_DIR
    start-yarn.sh
    start-dfs.sh
```


### Verifying Cluster is Running Properly

If you want to verify that everything is working you can use the `jps` command (Process ID on the left can be ignored):

Expected output from **NameNode**:
```console
    2912 NameNode
    3491 Jps
    3205 SecondaryNameNode
    3029 DataNode
    2489 NodeManager
    2380 ResourceManager
```

Expected output from each **DataNode**:
```console
    1221 NodeManager
    1578 Jps
    1387 DataNode
```

On **NameNode** you can use `hdfs dfsadmin -report` to view the status of HDFS. You can also access the UI for `Hadoop` and `Yarn` by going to:
- [Hadoop - Web UI](http://192.168.1.58:50070/)
- [Yarn - Web UI](http://192.168.1.58:8088/)


### Shutting Down The Cluster

To shut down the cluster we just need to run the stop scripts in reverse order. We start by shutting down yarn and finally dfs.
```console
    cd $HADOOP_CONF_DIR
    stop-yarn.sh
    stop-dfs.sh
```
