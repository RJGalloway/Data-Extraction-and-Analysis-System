
## Preparing NameNode

### Hadoop User Group & User Name

In this step, we'll create a new user group called `hadoop` and user called `hduser`

'''
    sudo addgroup hadoop
    sudo adduser --ingroup hadoop hduser
'''

> Press ENTER for all user information to select default; press Y to save

    sudo adduser hduser sudo


Everything Hadoop will be happening via the `hduser`. Let's change to this user.

    su hduser


### SSH Key

Although we are using a single node setup in this part, I decided to already create SSH keys. These will be the keys that the nodes use to talk to each other.

    cd ~
    mkdir .ssh
    ssh-keygen -t rsa -P ""


> Press ENTER to save the key to the default path `/home/hduser/.ssh/id_rsa`

    cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys


To verify that everything is working, you can easily create a SSH tunnel to localhost.

    ssh localhost


### Update `hosts`

Make sure you pick uniform names for all of your Pi nodes – in /etc/hostname, I used ‘RaspberryPiHadoopNameNode’ for the master, and ‘RaspberryPiHadoopDataNode#’ for the slaves. Let's update `hosts`:

    sudo nano /etc/hosts


Use the configuration below:

    #/etc/hosts
    127.0.0.1         localhost
    ::1               localhost ip6-localhost ip6-loopback
    ff02::1           ip6-allnodes
    ff02::2           ip6-allrouters

    127.0.0.1         raspberrypi-namenode

    192.168.1.*       RaspberryPiHadoopNameNode
    192.168.1.*       RaspberryPiHadoopDataNode1
    192.168.1.*       RaspberryPiHadoopDataNode2


### Installing Java8

Execute the following command to install Java8:

    sudo apt install openjdk-8-jre-headless openjdk-8-jdk-headless


Verify the installation with:

    java -version


You should see:

    openjdk version "11.0.9.1" 2020-11-04
    OpenJDK Runtime Environment (build 11.0.9.1+1-post-Raspbian-1deb10u2)
    OpenJDK Server VM (build 11.0.9.1+1-post-Raspbian-1deb10u2, mixed mode)


## Hadoop 2.9.2

### Installation

Follow the commands below to install Hadoop 2.9.2:

    wget https://archive.apache.org/dist/hadoop/core/hadoop-2.9.2/hadoop-2.9.2.tar.gz
    sudo tar -xvzf hadoop-2.9.2.tar.gz -C /opt/
    cd /opt
    sudo chown -R hduser:hadoop hadoop-2.9.2/

### Environment Variables

First we need to set a few environment variables. There are a few ways to do this, but I always do it by editing the .bashrc file.

    nano ~/.bashrc


Add the following lines at the end of the file:

    # Hadoop
    export JAVA_HOME=$(readlink -f /usr/ | sed "s:jre/bin/java::")
    export HADOOP_HOME=/opt/hadoop-2.9.2
    export HADOOP_MAPRED_HOME=$HADOOP_HOME
    export HADOOP_COMMON_HOME=$HADOOP_HOME
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
    export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib"
    export HADOOP_HDFS_HOME=$HADOOP_HOME
    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
    export YARN_HOME=$HADOOP_HOME
    export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
    export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

      # Suppress message: "WARN util.NativeCodeLoader: Unable to load
      # native-hadoop library for your platform... using builtin-java
      # classes where applicable"

    # Hadoop Warning Suppression
    export HADOOP_HOME_WARN_SUPPRESS=1
    export HADOOP_ROOT_LOGGER="WARN,DRFA"

Changes in the .bashrc are not applied when you save this file. Refresh the environment variables by typing:

    source ~/.bashrc

If everything is configured right, you should be able to print the installed version of Hadoop by typing:

    hadoop version

Expected output:

    Hadoop 2.9.2
    Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r 826afbeae31ca687bc2f8471dc841b66ed2c6704
    Compiled by ajisaka on 2018-11-13T12:42Z
    Compiled with protoc 2.5.0
    From source with checksum 3a9939967262218aa556c684d107985
    This command was run using /opt/hadoop-2.9.2/share/hadoop/common/hadoop-common-2.9.2.jar


## Configurations

Begin by running the following command line in terminal to determine where you Java home path is. You can find the directory by typing `whereis java`.

Expected output:

    java: /usr/bin/java /usr/share/java /usr/share/man/man1/java.1.gz


Next, let's go to the directory that contains all the configuration files of Hadoop. We want to edit the _hadoop-env.sh_ file. For some reason we need to configure JAVA_HOME manually in this file, Hadoop seems to ignore our $JAVA_HOME.

    cd $HADOOP_CONF_DIR
    nano hadoop-env.sh


Look for the line saying `JAVA_HOME=${JAVA_HOME}` and change it to your Java install directory. This was how the line looked after I changed it:

    export JAVA_HOME=/usr


There are quite a few files that need to be edited now. These are XML files, you just have to paste the code bits below between the _configuration_ tags.

**core-site.xml**

    nano core-site.xml


Add the following pair/key between `<configuration>` and `</configuration>`:

    <property>  
      <name>fs.default.name</name>
      <value>hdfs://RaspberryPiHadoopNameNode:54310</value>
    </property>  
    <property>  
      <name>hadoop.tmp.dir</name>
      <value>/hdfs/tmp</value>
    </property>


**hdfs-site.xml**

    nano hdfs-site.xml


Add the following pair/key between `<configuration>` and `</configuration>`:

    <property>  
      <name>dfs.replication</name>  
      <value>1</value>  
    </property>


Copy template to mapred-site.xml

    cp mapred-site.xml.template mapred-site.xml


**mapred-site.xml**

    nano mapred-site.xml


Add the following pair/key between `<configuration>` and `</configuration>`:

    <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>
    <property>
      <name>mapreduce.map.memory.mb</name>
      <value>256</value>
    </property>
    <property>
      <name>mapreduce.map.java.opts</name>
      <value>-Xmx210m</value>
    </property>
    <property>
      <name>mapreduce.reduce.memory.mb</name>
      <value>256</value>
    </property>
    <property>
      <name>mapreduce.reduce.java.opts</name>
      <value>-Xmx210m</value>
    </property>
    <property>
      <name>yarn.app.mapreduce.am.resource.mb</name>
      <value>256</value>
    </property>


* The first property tells us that we want to use Yarn as the MapReduce framework. The other properties are some specific settings for our Raspberry Pi. For example we tell that the Yarn Mapreduce Application Manager gets 256 megabytes of RAM and so does the Map and Reduce containers. These values allow us to actually run stuff, the default size is 1,5GB which our Pi can't deliver with its 1GB RAM.


**yarn-site.xml**

    nano yarn-site.xml


Add the following pair/key between `<configuration>` and `</configuration>`:

    <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
    </property>
    <property>
      <name>yarn.nodemanager.resource.cpu-vcores</name>
      <value>4</value>
    </property>
    <property>
      <name>yarn.nodemanager.resource.memory-mb</name>
      # <!-- <value>768</value> -->
      <value>1024</value>
    </property>
    <property>
      <name>yarn.scheduler.minimum-allocation-mb</name>
      <value>128</value>
    </property>
    <property>
      <name>yarn.scheduler.maximum-allocation-mb</name>
      # <!-- <value>768</value> -->
      <value>1024</value>
    </property>
    <property>
      <name>yarn.scheduler.minimum-allocation-vcores</name>
      <value>1</value>
    </property>
    <property>
      <name>yarn.scheduler.maximum-allocation-vcores</name>
      <value>4</value>
    </property>

    # <!-- https://stackoverflow.com/questions/20598513/only-one-node-in-resourcemanager -->

    <property>
      <name>yarn.resourcemanager.scheduler.address</name>
      <value>RaspberryPiHadoopNameNode:8030</value>
    </property>
    <property>
      <name>yarn.resourcemanager.address</name>
      <value>RaspberryPiHadoopNameNode:8032</value>
    </property>
    <property>
      <name>yarn.resourcemanager.webapp.address</name>
      <value>RaspberryPiHadoopNameNode:8088</value>
    </property>
    <property>
      <name>yarn.resourcemanager.resource-tracker.address</name>
      <value>RaspberryPiHadoopNameNode:8031</value>
    </property>
    <property>
      <name>yarn.resourcemanager.admin.address</name>
      <value>RaspberryPiHadoopNameNode:8033</value>
    </property>

    # <!-- Values Added Below by DQYDJ -->

    <property>
      <name>yarn.nodemanager.vmem-check-enabled</name>
      <value>false</value>
      <description>Whether virtual memory limits will be enforced for containers</description>
    </property>
    <property>
      <name>yarn.nodemanager.vmem-pmem-ratio</name>
      <value>4</value>
      <description>Ratio between virtual memory to physical memory when setting memory limits for containers</description>
    </property>


* This file tells Hadoop some information about this node, like the maximum number of memory and cores that can be used. We limit the usable RAM to 768 megabytes, that leaves a bit of memory for the OS and Hadoop. A container will always receive a memory amount that is a multitude of the minimum allocation, 128 megabytes. For example a container that needs 450 megabytes, will get 512 megabytes assigned.


**slaves**

    nano slaves


Remove `localhost` and add both the NameNode and all the DataNodes:

    RaspberryPiHadoopNameNode
    RaspberryPiHadoopDataNode1
    RaspberryPiHadoopDataNode2


* Two files must be edited on the master only: slaves and masters. The slaves file tells the master node which other nodes can be used for this cluster. Just add the nodes that you want to use for data processing to this file, perhaps even including the master node itself.

Make sure that your system can resolve these hostnames. This file only goes on the master node, the other nodes don't need it.


## HDFS

### Preparing HDFS on NameNode

Run the following commands to setup HDFS:

    sudo mkdir -p /hdfs/tmp
    sudo chown hduser:hadoop /hdfs/tmp
    chmod 750 /hdfs/tmp
    hdfs namenode -format


You may see the error, you can ignore it:

    WARNING: An illegal reflective access operation has occurred
    WARNING: Illegal reflective access by org.apache.hadoop.security.authentication.util.KerberosUtil (file:/opt/hadoop-2.9.2/share/hadoop/common/lib/hadoop-auth-2.9.2.jar) to method sun.security.krb5.Config.getInstance()
    WARNING: Please consider reporting this to the maintainers of org.apache.hadoop.security.authentication.util.KerberosUtil
    WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
    WARNING: All illegal access operations will be denied in a future release


### Testing HDFS on NameNode

You can try testing HDFS on Namenode to ensure you can access it.

    start-dfs.sh


You'll notice that it hangs, this is because we've listed `RaspberryPiHadoopDataNode1` and `RaspberryPiHadoopDataNode2` which haven't been setup yet. Just press `Ctrl + C` to quit the command. Run the following command:

    hdfs dfs -ls /


If no errors return, you have successfully run your first HDFS command.
