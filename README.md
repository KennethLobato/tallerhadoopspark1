# tallerhadoopspark1
***

1. [Preparing the environment](#Preparing the environment)
---
Download the code from this repository:
```Bash
git clone https://github.com/KennethLobato/tallerhadoopspark1.git
```

## Preparing the environment
Launch vagrant:
```Bash
vagrant up
```

Wait for vagrant to finish the setup of this small cluster, and connect to both machines "ubuntu1" and "ubuntu2" in two separate command shells:
```Bash
vagrant ssh ubuntu1
```
```Bash
vagrant ssh ubuntu2
```

Verify that the /etc/hosts has the following information (otherwise edit it with vim or nano):
```Bash
vagrant@ubuntu1:~$ cat /etc/hosts
127.0.0.1 localhost

10.0.0.10 ubuntu1.tallerhadoop1.org ubuntu1
10.0.0.11 ubuntu2.tallerhadoop1.org ubuntu2
```

Connect to the following machines with SSH to accept their fingerprint:
```Bash
vagrant@ubuntu1:~$ ssh ubuntu1
vagrant@ubuntu1:~$ ssh ubuntu2
vagrant@ubuntu1:~$ ssh localhost
vagrant@ubuntu1:~$ ssh 0.0.0.0
```

Ask to format the HDFS's NameNode:
```Bash
vagrant@ubuntu1:~$ hdfs namenode -format
```

Start the HDFS:
```Bash
vagrant@ubuntu1:~$ cd /usr/local/hadoop/
vagrant@ubuntu1:~$ ./sbin/start-dfs.sh
```
Verify that the namenode, datanode and snamenode are started properly:
```Bash
vagrant@ubuntu1:/usr/local/hadoop$ jps
4260 SecondaryNameNode
4039 DataNode
4376 Jps
3852 NameNode
```

```Bash
vagrant@ubuntu2:~$ jps
3625 Jps
3550 DataNode
```

Create the folders /user/vagrant in the HDFS:
```Bash
vagrant@ubuntu1:/usr/local/hadoop$ hdfs dfs -mkdir -p /user/vagrant/
vagrant@ubuntu1:/usr/local/hadoop$ hdfs dfs -ls /
Found 1 items
drwxr-xr-x   - vagrant supergroup          0 2016-10-12 17:02 /user
vagrant@ubuntu1:/usr/local/hadoop$ hdfs dfs -ls /user/
Found 1 items
drwxr-xr-x   - vagrant supergroup          0 2016-10-12 17:02 /user/vagrant
vagrant@ubuntu1:/usr/local/hadoop$ hdfs dfs -ls /user/vagrant/
```

Start YARN Resource Manager:
```Bash
vagrant@ubuntu1:/usr/local/hadoop$ ./sbin/start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop-2.7.3/logs/yarn-vagrant-resourcemanager-ubuntu1.out
ubuntu2: starting nodemanager, logging to /usr/local/hadoop-2.7.3/logs/yarn-vagrant-nodemanager-ubuntu2.out
ubuntu1: starting nodemanager, logging to /usr/local/hadoop-2.7.3/logs/yarn-vagrant-nodemanager-ubuntu1.out
```
Verify that the following processes are running (ResourceManager and NodeManager):
```Bash
vagrant@ubuntu1:/usr/local/hadoop$ jps
4592 ResourceManager
4756 NodeManager
4260 SecondaryNameNode
4039 DataNode
5049 Jps
3852 NameNode
```
```Bash
vagrant@ubuntu2:~$ jps
3729 NodeManager
3833 Jps
3550 DataNode
```

Launch Spark in Standalone cluster mode:
```Bash
vagrant@ubuntu1:/usr/local/hadoop$ cd /usr/local/spark/
vagrant@ubuntu1:/usr/local/spark$ ./sbin/start-master.sh
vagrant@ubuntu1:/usr/local/spark$ ./sbin/start-slave.sh spark://ubuntu1:7077
```

```Bash
vagrant@ubuntu2:/usr/local/spark$ ./sbin/start-slave.sh spark://ubuntu1:7077
```

Make a test that spark is properly installed (this should launch the shell REPL):
```Bash
vagrant@ubuntu1:/usr/local/spark$ ./bin/spark-shell --master spark://ubuntu1:7077
(...)
scala> sc
res1: org.apache.spark.SparkContext = org.apache.spark.SparkContext@4045fd1f

scala> sqlContext
res2: org.apache.spark.sql.SQLContext = org.apache.spark.sql.hive.HiveContext@738a977d
```

Open in an explorer the following addresses:
- ubuntu1:50070 (HDFS)
- ubuntu1:8088 (YARN)
- ubuntu1:8080 (Spark Standalone Cluster)

We can set up the level of verbosity with the following commands:

```Bash
vagrant@ubuntu1:/usr/local/spark$ mv conf/log4j.properties.template conf/log4j.properties
```

Editar el fichero y cambiar las líneas:
```Bash
log4j.rootCategory=INFO, console
```

Por:
```Bash
log4j.rootCategory=ERROR, console
```

Checking the spark Standalone cluster is running property with some examples:
```Bash
vagrant@ubuntu1:/usr/local/spark$ ./bin/spark-submit examples/src/main/python/pi.py 10
Pi is roughly 3.140388
```

Follow up the examples guide of Spark and take a look to the spark-submit command:
- http://spark.apache.org/docs/1.6.2/quick-start.html
- http://spark.apache.org/docs/latest/submitting-applications.html

NOTES:
- To install sbt:
```Bash
vagrant@ubuntu1:/usr/local/spark$ wget https://dl.bintray.com/sbt/debian/sbt-0.13.12.deb
vagrant@ubuntu1:/usr/local/spark$ sudo dpkg -i sbt-0.13.12.deb
vagrant@ubuntu1:/usr/local/spark$ sudo apt-get install scala
```

ZEPPELIN
Launch Zeppelin

- Edit file zeppelin-site.xml.template and change port 8080 to 9090

```Bash
vagrant@ubuntu1:/usr/local/zeppelin$ conf/zeppelin-site.xml.template conf/zeppeling-site.xml
vagrant@ubuntu1:/usr/local/zeppelin$ ./bin/zeppelin-daemon start
Zeppelin is running                                        [  OK  ]
vagrant@ubuntu1:/usr/local/zeppelin$ ./bin/zeppelin-daemon status
Zeppelin is running                                        [  OK  ]
```
