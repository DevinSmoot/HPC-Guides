## Set up Hadoop on the Cluster

**This section is incomplete**

##### Step 1 - Create new user and group

Create hduser and hadoop group:

```
sudo addgroup HADOOP
sudo adduser --ingroup hadoop hduser
sudo usermod -a -G sudo hduser
```

**_Note:_** All other actions from now on must be completed under the *hduser* account
Exit out and login as *hduser*

##### Step 2 - Disable IPv6

Edit /etc/sysctl.conf:

``sudo nano /etc/sysctl.conf``

Add the following lines to the end of the file:

```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

Apply changes:

``sudo sysctl -p``


##### Step 3 - Download latest Apache Hadoop

```
cd ~
wget http://apache.osuosl.org/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz
```

Extract files:

```
tar xvfz hadoop-2.7.3.tar.gz
```

Create Hadoop folder under /usr/local:

```
sudo mkdir /usr/local/hadoop
```

Move the Hadoop folder to /usr/local/hadoop:

```
sudo mv hadoop-2.7.3 /usr/local/hadoop
sudo chown hduser:hadoop -R /usr/local/hadoop
```

Create Hadoop temp directories for Namenode and Datanode:

```
sudo mkdir -p /usr/local/hadoop_tmp/hdfs/namenode
sudo mkdir -p /usr/local/hadoop_tmp/hdfs/datanode
sudo chown hduser:hadoop -R /usr/local/hadoop_tmp
```

##### Step 4 - Update Hadoop Configuration Files

Update $HOME/.bashrc

``sudo nano ~/.bashrc``

Add environment variables to .bashrc file:

```
# -- HADOOP ENVIRONMENT VARIABLES START -- #
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-armhf
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
# -- HADOOP ENVIRONMENT VARIABLES END -- #
```

Change to the /usr/local/hadoop/etc/hadoop directory:

``cd /usr/local/hadoop/etc/hadoop``

Edit hadoop-env.sh:

``sudo nano hadoop-env.sh``

Change the following line:

``JAVA_HOME=``

To:

``JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-armhf``

Edit core-site.xml:
