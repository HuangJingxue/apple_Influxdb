
--保存策略删除会导致measurements清除
```shell
> create database mytest

> show measurements
>


> insert info,ip=172.17.1.1,hostname=DB,OS=CentOS cpu_usage=77.77,mem_usage=7.777,sum_conn=77i
> insert info,ip=172.17.1.2,hostname=DB,OS=CentOS cpu_usage=66.66,mem_usage=6.666,sum_conn=66i
> insert info,ip=172.17.1.3,hostname=OA,OS=Windows cpu_usage=55.55,mem_usage=5.555,sum_conn=55i
> insert info,ip=172.17.1.1,hostname=DB,OS=CentOS cpu_usage=44.77,mem_usage=4.777,sum_conn=47i
> insert info,ip=172.17.1.2,hostname=DB,OS=CentOS cpu_usage=33.66,mem_usage=3.666,sum_conn=36i
> insert info,ip=172.17.1.3,hostname=OA,OS=Windows cpu_usage=22.55,mem_usage=2.555,sum_conn=25i
>


> select * from info
> name: info
> time                OS      cpu_usage hostname ip         mem_usage sum_conn
> ----                --      --------- -------- --         --------- --------
> 1582619436829745192 CentOS  60.33     DB       172.17.1.1 6.453     99
> 1582788256292315622 CentOS  45.44     DB       172.17.1.2 4.753     88
> 1582788334812734722 Windows 45.54     OA       172.17.1.3 3.456     175
> 1582788654066463878 CentOS  77.77     DB       172.17.1.1 7.777     77
> 1582788654073687401 CentOS  66.66     DB       172.17.1.2 6.666     66
> 1582788654077510054 Windows 55.55     OA       172.17.1.3 5.555     55
> 1582788654091570321 CentOS  44.77     DB       172.17.1.1 4.777     47
> 1582788654096323714 CentOS  33.66     DB       172.17.1.2 3.666     36
> 1582788654820469352 Windows 22.55     OA       172.17.1.3 2.555     25

> show field keys from info
> name: info
> fieldKey  fieldType
> --------  ---------
> cpu_usage float
> mem_usage float
> sum_conn  integer



> insert test2,col1=c_1,col2=c_2 value=10i,value2=T
> insert test3,col1=c_1,col2=c_2 value=10,value2=T
> show field keys from test2
> name: test2
> fieldKey fieldType
-------- ---------
value    integer
value2   boolean

> show field keys from test3
> name: test3
> fieldKey fieldType
-------- ---------
value    float
value2   boolean

> show measurements
> name: measurements
> name
----
info
test2
test3
> show retention policies on "mytest"
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 0s       168h0m0s           1        true
> exit

#删除保存策略
> [root@influx backup]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> use mytest
> Using database mytest
> drop retention policy "autogen" on "mytest"
> show measurements  #measurement被清除

正确步骤：待验证？
先创建一个其他的保存策略，并设置为默认。
再清除当前的保存策略
> exit
```

--备份恢复数据
```shell
# 备份
> [root@influx backup]# influxd backup -portable -database mytest /root/backup/mytest

> 2020/02/27 14:39:02 backing up metastore to /root/backup/mytest/meta.00
> 2020/02/27 14:39:02 backing up db=mytest
> 2020/02/27 14:39:02 backing up db=mytest rp=autogen shard=3 to /root/backup/mytest/mytest.autogen.00003.00 since 0001-01-01T00:00:00Z
> 2020/02/27 14:39:02 backup complete:
> 2020/02/27 14:39:02     /root/backup/mytest/20200227T063902Z.meta
> 2020/02/27 14:39:02     /root/backup/mytest/20200227T063902Z.s3.tar.gz
> 2020/02/27 14:39:02     /root/backup/mytest/20200227T063902Z.manifest


> [root@influx backup]# cd mytest
> [root@influx mytest]# ll
> total 12
> -rw------- 1 root root  293 Feb 27 14:39 20200227T063902Z.manifest
> -rw-r--r-- 1 root root  241 Feb 27 14:39 20200227T063902Z.meta
> -rw------- 1 root root 1036 Feb 27 14:39 20200227T063902Z.s3.tar.gz


> [root@influx mytest]# file 20200227T063902Z.manifest
> 20200227T063902Z.manifest: ASCII text
> [root@influx mytest]# head -n 10 20200227T063902Z.manifest
> {
> "meta": {
> "fileName": "20200227T063902Z.meta",
> "size": 236
> },
> "limited": false,
> "files": [
> {
>  "database": "mytest",
>  "policy": "autogen",
> [root@influx mytest]# head -n 100 20200227T063902Z.manifest
> {
> "meta": {
> "fileName": "20200227T063902Z.meta",
> "size": 236
> },
> "limited": false,
> "files": [
> {
>  "database": "mytest",
>  "policy": "autogen",
>  "shardID": 3,
>  "fileName": "20200227T063902Z.s3.tar.gz",
>  "size": 3584,
>  "lastModified": 0
> }
> ]
> }

#恢复所有数据库
> [root@influx backup]# influxd restore -portable /root/backup/mytest

#恢复到指定数据库
> [root@influx backup]# influxd restore -portable -db mytest /root/backup/mytest

#恢复到已存在的数据库
1.先恢复到临时库mytest1
> [root@influx backup]# influxd restore -portable -db mytest -newdb mytest1 /root/backup/mytest
> 2020/02/27 15:40:25 Restoring shard 8 live from backup 20200227T073613Z.s8.tar.gz
> [root@influx backup]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> use  mytest1
> Using database mytest1
> show measurements
> name: measurements
> name
----
info
test2
test3
> show databases
> name: databases
> name
----
_internal
mytest
mytest1

> use mytest1
> Using database mytest1

2.通过select..into方式将临时数据库的数据加载到原数据库当中
存在问题，所有数字类型都会成为浮点型。
> select * into mytest..:measurement from /.*/ group by *
> ERR: retention policy not found: autogen

由于原数据中保存策略被误删除，导致无法使用加载数据，处理方式：
a.根据恢复出来的新库，创建一个保留策略
b.再将临时数据库数据加载到原库即可

> use mytest
> Using database mytest

> create retention policy "autogen" on "mytest" duration 7d replication 1 default

保留策略创建待研究？

> use mytest2
> Using database mytest2
> select * into mytest..:measurement from /.*/ group by *
> name: result
> time written
---- -------
0    11

> use mytest
> Using database mytest

> show measurements
> name: measurements
> name
----
info
test2
test3
> select * from info;
> name: info
> time                OS      cpu_usage hostname ip         mem_usage sum_conn
> ----                --      --------- -------- --         --------- --------
> 1582619436829745192 CentOS  60.33     DB       172.17.1.1 6.453     99
> 1582788256292315622 CentOS  45.44     DB       172.17.1.2 4.753     88
> 1582788334812734722 Windows 45.54     OA       172.17.1.3 3.456     175
> 1582788654066463878 CentOS  77.77     DB       172.17.1.1 7.777     77
> 1582788654073687401 CentOS  66.66     DB       172.17.1.2 6.666     66
> 1582788654077510054 Windows 55.55     OA       172.17.1.3 5.555     55
> 1582788654091570321 CentOS  44.77     DB       172.17.1.1 4.777     47
> 1582788654096323714 CentOS  33.66     DB       172.17.1.2 3.666     36
> 1582788654820469352 Windows 22.55     OA       172.17.1.3 2.555     25
> select * from test2
> name: test2
> time                col1 col2 value value2
> ----                ---- ---- ----- ------
> 1582788851836670999 c_1  c_2  10    true
> select * from test3
> name: test3
> time                col1 col2 value value2
> ----                ---- ---- ----- ------
> 1582788859956575890 c_1  c_2  10    true

> show field keys from test2
> name: test2
> fieldKey fieldType
-------- ---------
value    integer
value2   boolean
> show field keys from test3
> name: test3
> fieldKey fieldType
-------- ---------
value    float
value2   boolean

```
--查询区分大小写
查询：
tag key 使用双引号或者不用
tag values 使用单引号
field 使用单引号

插入：
insert 使用双引号

```shell

[root@influx ~]# influx
Connected to http://localhost:8086 version 1.7.10
InfluxDB shell version: 1.7.10
> show databases
> name: databases
> name
----
_internal
mytest

> show tag values from info with key="OS"
> name: info
> key value
--- -----
OS  CentOS
OS  Windows
> show tag values from info with key=OS
> name: info
> key value
--- -----
OS  CentOS
OS  Windows
> show tag values from info with key=os


> show tag values from info with key in (OS,hostname)
> name: info
> key      value
> ---      -----
> OS       CentOS
> OS       Windows
> hostname DB
> hostname OA

> show tag values from info with key in (OS,hostname) where ip='172.17.1.1'
> name: info
> key      value
> ---      -----
> OS       CentOS
> hostname DB



> show tag values from info with key in (OS,hostname) where mem_usage=6.453
> name: info
> key      value
> ---      -----
> OS       CentOS
> OS       Windows

> show tag values from info with key in (ip,hostname) where OS=CentOS
> show tag values from info with key in (ip,hostname) where OS='CentOS'
> name: info
> key      value
> ---      -----
> hostname DB
> ip       172.17.1.1
> ip       172.17.1.2
> show series from info
> key
---
info,OS=CentOS,hostname=DB,ip=172.17.1.1
info,OS=CentOS,hostname=DB,ip=172.17.1.2
info,OS=Windows,hostname=OA,ip=172.17.1.3
> EXIT
```
--备份出的数据是float类型，源表是int类型提示报错
```shell
> select * INTO mytest..:MEASUREMENT from /.*/ GROUP BY *
> ERR: partial write: field type conflict: input field "value" on measurement "test2" is type float, already exists as type integer dropped=1
> use mytest
> Using database mytest
> select * from test2
> name: test2
> time                col1 col2 value value2
> ----                ---- ---- ----- ------
> 1582616304173221269 c1   c2   10    true

> show field keys from test2
> name: test2
> fieldKey fieldType
-------- ---------
value    integer
value2   boolean
> use mytest1
> Using database mytest1
> show measurements
> name: measurements
> name
----
info
test2
test3
test4
test_1
test_cpu
> show field keys from test2
> name: test2
> fieldKey fieldType
-------- ---------
value    integer
value2   boolean


> select * INTO mytest..:MEASUREMENT from /.*/ GROUP BY *
> ERR: partial write: field type conflict: input field "value" on measurement "test2" is type float, already exists as type integer dropped=1
> select * from test2
> name: test2
> time                col1 col2 value value2
> ----                ---- ---- ----- ------
> 1582616304173221269 c1   c2   10    true
> use mytest
> Using database mytest

# 将原库test2删除
> drop measurement test2


> select * INTO mytest..:MEASUREMENT from /.*/ GROUP BY *
> name: result
> time written
---- -------
0    13
> use mytest

> select * from test2
> name: test2
> time                col1 col2 value value2
> ----                ---- ---- ----- ------
> 1582616304173221269 c1   c2   10    true
> show field keys from test2
> name: test2
> fieldKey fieldType
-------- ---------
value    float
value2   boolean
> exit
```
--保留策略恢复
> 通过验证采用influxd restore恢复到新库中可以。
> 采用select..into方式不起作用，待研究?
```shell

> [root@influx mytest]# influxd restore -portable -db mytest -newdb mytest2 -rp autogen -newrp autogen_bak /root/backup/mytest
> 2020/02/27 15:04:51 Restoring shard 3 live from backup 20200227T063902Z.s3.tar.gz
> [root@influx mytest]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> use mytest2
> Using database mytest2
> show retention policies on mytest2
> name        duration shardGroupDuration replicaN default
> ----        -------- ------------------ -------- -------
> autogen_bak 0s       168h0m0s           1        true
> select * into mytest.autogen.:measurement from /mytest2.autogen_bak.*/ group by *
> name: result
> time written
---- -------
0    0
> use mytest
> Using database mytest
> show retention policies on mytest
> name duration shardGroupDuration replicaN default
---- -------- ------------------ -------- -------

> use mytest
> Using database mytest
> create retention policy "autogen_new" on "mytest" duration 3w replication 1 default
> show retention policies on mytest
> name        duration shardGroupDuration replicaN default
> ----        -------- ------------------ -------- -------
> autogen_new 504h0m0s 24h0m0s            1        true

> use mytest2
> Using database mytest2
> select * into mytest.autogen.:measurement from /mytest2.autogen_bak.*/ group by *
> name: result
> time written
---- -------
0    0

> use mytest
> Using database mytest
> create retention policy "autogen" on "mytest" duration 3w replication 1 default
> use mytest2
> Using database mytest2
> select * into mytest.autogen.:measurement from /mytest2.autogen_bak.*/ group by *
> name: result
> time written
---- -------
0    0
> show retention policies on mytest
> name        duration shardGroupDuration replicaN default
> ----        -------- ------------------ -------- -------
> autogen_new 504h0m0s 24h0m0s            1        false
> autogen     504h0m0s 24h0m0s            1        true
> exit


> show retention policies on mytest
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 504h0m0s 24h0m0s            1        true
> alter retention policy "autogen_bak" on "test4" duration 3d default
> ERR: database not found: test4
> alter retention policy "autogen_bak" on "mytest4" duration 3d default
> show retention policies on mytest4
> name        duration shardGroupDuration replicaN default
> ----        -------- ------------------ -------- -------
> autogen_bak 72h0m0s  24h0m0s            1        true
> show retention policies on mytest
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 504h0m0s 24h0m0s            1        true
> select * into mytest.autogen.:measurement from /mytest4.autogen_bak.*/ group by *
> name: result
> time written
---- -------
0    0
> show retention policies on mytest
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 504h0m0s 24h0m0s            1        true
> select * into mytest..:measurement from /.*/ group by *
> name: result
> time written
---- -------
0    11
> show retention policies on mytest
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 504h0m0s 24h0m0s            1        true
>
```
