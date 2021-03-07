###### YARN on a Single Node

Configure parameters as follows:`etc/hadoop/mapred-site.xml`:

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

`etc/hadoop/yarn-site.xml`:

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

Start ResourceManager daemon and NodeManager daemon:

```
  $ sbin/start-yarn.sh
```

启动后jps查看进程，并进入管理页面

```
http://外网访问ip:8088/cluster
```

>==如果存在无法访问的情况， 可以先用一下命令检查防火墙是否开启==

```
telnet 外网ip 端口号
```

>很多有技术的运维人员，都会把机器设置禁止ping，且会封25号端口
>
>如果ping不同可以尝试telnet

###### 案例

检查hadoop 环境的环境变量(环境配置最好配置在~/.bashrc)

```
export HADOOP_HOME=/home/hadoop/app/hadoop
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH

source .bashrc
```

在~/data/input/下造test数据1.log

在hdfs下创建文件夹

```
hdfs dfs -mkdir -p /wordcount/input
```

把测试文件传入hdfs

```
hdfs dfs -put 1.log /wordcount/input
```

查看当前hdfs目录

```
hdfs dfs -ls hdfs://ruozedata001:9000/
hdfs dfs -ls /
# 这两行代码是一样的,只不过前面省略了
```

执行以下代码

```
hadoop jar ./share/hadoop/mapreduce2/hadoop-mapreduce-examples-2.6.0-cdh5.7.0.jar wordcount /wordcount/input/ /wordcount/output1/
```

==为了防止被挖矿==

可以用top命令查看cpu和内存使用率

并且可以用kill -9 命令杀进程如果过一会儿还重新启动可以判断是被挖矿

所以要修改yarn的默认端口

调整8088端口,在etc/hadoop/yarn-site.xml 新增:

```
<property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>${yarn.resourcemanager.hostname}:7776</value>
</property>
```

重启yarn服务

```
$ sbin/stop-yarn.sh
sbin/start-yarn.sh
```

jps 查询进程号, 用netstat命令查询端口号

```
netstat -nlp | grep 进程号
```

```
tcp6  0  0 :::7776  :::*   LISTEN   25687/java  
```

更改成功

查找jps对应的标识文件目录

```
/tmp/hsperfdata_hadoop/
```

>此处是根目录的tmp文件夹不是当前用户目录下的

### **特别注意**

==当看见 process information unavailable==
==不能代表进程是存在 或者不存在，要当心，尤其使用jps命令来做脚本状态检测的==
==一般使用经典的 ps -ef | grep xxx命令去查看进程是否存在，==
==这才是真正的状态检测。==

==但是: 比如spark thriftserver +hive 会启动一个driver 进程 110，==
==默认端口号 10001。由于该程序的内存泄露或者某种bug，导致==
==进程ps是存在的，10001端口号下线了，就不能够对外提供服务。==

==总结: 未来做任何程序的状态检测，必须通过端口号来。==



###### /tmp 目录下的pid 问题

Pid文件是不可以删除的, 会影响启动或者重启

因为在启动的脚本里, 会创建pid文件, 关闭时候会查找相应的pid文件,

如果在关闭前删除pid文件, 就会使关闭脚本找不到pid文件, 默认已经关闭,实际进程还在运行

总结: 如果pid放在tmp目录30天以后被清理掉, 那么脚本就不能正常关闭服务了

解决办法:

移动到当前用户自己创建的tmp目录下

修改hadoop-env.sh

````
export HADOOP_PID_DIR=/home/hadoop/tmp
````

重启

```
sh stop-dfs.sh && sh start-dfs.sh
```

查看 ~/tmp目录下生成新的pid文件

###### namenode的作用

==维护一个文件被切割哪些块, 这些块被存放到哪些机器==

==生产上要尽量避免小文件的hdfs上的存储==

- 数据传输到hdfs之前,提前合并
- 数据已经到hdfs, 就定时的业务低谷期,去合并冷文件







**作业:**

**了解split切分的规则**

**map reduce 的task的分配规则**

