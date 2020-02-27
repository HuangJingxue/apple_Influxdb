
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
login as: root
root@172.10.1.5's password:
Last login: Tue Feb 25 14:47:38 2020 from 172.10.1.1
[root@influx ~]# influx
Connected to http://localhost:8086 version 1.7.10
InfluxDB shell version: 1.7.10
> show databases
> name: databases
> name
----
_internal
mytest
> use mytest
> Using database mytest
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
> select * from info
> name: info
> time                OS      cpu_usage hostname ip         mem_usage sum_conn
> ----                --      --------- -------- --         --------- --------
> 1582619436829745192 CentOS  60.33     DB       172.17.1.1 6.453     99
> 1582619583109758294 CentOS  45.44     DB       172.17.1.2 4.753     88
> 1582622294550197570 Windows 45.54     OA       172.17.1.3 3.456     175
> 1582623206783512818 CentOS  77.77     DB       172.17.1.1 7.777     77
> 1582623229806491676 CentOS  66.66     DB       172.17.1.2 6.666     66
> 1582623263478460723 Windows 55.55     OA       172.17.1.3 5.555     55
> 1582623281958524964 CentOS  44.77     DB       172.17.1.1 4.777     47
> 1582623290974590605 CentOS  33.66     DB       172.17.1.2 3.666     36
> 1582623300598878112 Windows 22.55     OA       172.17.1.3 2.555     25
>
> show series from info
> key
---
info,OS=CentOS,hostname=DB,ip=172.17.1.1
info,OS=CentOS,hostname=DB,ip=172.17.1.2
info,OS=Windows,hostname=OA,ip=172.17.1.3
> select time,os from info
> select sum_conn,cpu_usage from info
> name: info
> time                sum_conn cpu_usage
> ----                -------- ---------
> 1582619436829745192 99       60.33
> 1582619583109758294 88       45.44
> 1582622294550197570 175      45.54
> 1582623206783512818 77       77.77
> 1582623229806491676 66       66.66
> 1582623263478460723 55       55.55
> 1582623281958524964 47       44.77
> 1582623290974590605 36       33.66
> 1582623300598878112 25       22.55
> show tag keys from info
> name: info
> tagKey
------
OS
hostname
ip
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
> show tag values from info with key=OS
> name: info
> key value
--- -----
OS  CentOS
OS  Windows

> show tag values from info with key=hostname
> name: info
> key      value
> ---      -----
> hostname DB
> hostname OA

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

# 大小写区分
> show tag values from info with key=os
> show series from info
> key
---
info,OS=CentOS,hostname=DB,ip=172.17.1.1
info,OS=CentOS,hostname=DB,ip=172.17.1.2
info,OS=Windows,hostname=OA,ip=172.17.1.3



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
> hostname DB
> hostname OA
> show tag values from info with key in (OS,hostname) where sum_conn=55
> name: info
> key      value
> ---      -----
> OS       CentOS
> OS       Windows
> hostname DB
> hostname OA
> show tag values from info with key in (ip,hostname) where sum_conn=55
> name: info
> key      value
> ---      -----
> hostname DB
> hostname OA
> ip       172.17.1.1
> ip       172.17.1.2
> ip       172.17.1.3
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


> [root@influx ~]# mkdir backup
> [root@influx ~]# cd backup/
> [root@influx backup]# influxd backup -database mytest ./mytest_all.bak
> 2020/02/27 13:43:58 backing up metastore to mytest_all.bak/meta.00
> 2020/02/27 13:43:58 backing up db=mytest
> 2020/02/27 13:43:58 backing up db=mytest rp=autogen shard=3 to mytest_all.bak/my                                                                                        test.autogen.00003.00 since 0001-01-01T00:00:00Z
> 2020/02/27 13:43:58 backup complete:
> 2020/02/27 13:43:58     mytest_all.bak/mytest_all.bak/meta.00
> 2020/02/27 13:43:58     mytest_all.bak/mytest_all.bak/mytest.autogen.00003.00

> [root@influx meta]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> show databases
> name: databases
> name
----
_internal
mytest
> use mytest
> Using database mytest

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
> select * from info
> name: info
> time                OS      cpu_usage hostname ip         mem_usage sum_conn
> ----                --      --------- -------- --         --------- --------
> 1582619436829745192 CentOS  60.33     DB       172.17.1.1 6.453     99
> 1582619583109758294 CentOS  45.44     DB       172.17.1.2 4.753     88
> 1582622294550197570 Windows 45.54     OA       172.17.1.3 3.456     175
> 1582623206783512818 CentOS  77.77     DB       172.17.1.1 7.777     77
> 1582623229806491676 CentOS  66.66     DB       172.17.1.2 6.666     66
> 1582623263478460723 Windows 55.55     OA       172.17.1.3 5.555     55
> 1582623281958524964 CentOS  44.77     DB       172.17.1.1 4.777     47
> 1582623290974590605 CentOS  33.66     DB       172.17.1.2 3.666     36
> 1582623300598878112 Windows 22.55     OA       172.17.1.3 2.555     25
> show series from infp
> show series from info
> key
---
info,OS=CentOS,hostname=DB,ip=172.17.1.1
info,OS=CentOS,hostname=DB,ip=172.17.1.2
info,OS=Windows,hostname=OA,ip=172.17.1.3


> show tag keys from info
> name: info
> tagKey
------
OS
hostname
ip

> show tag values from info with key=ip
> name: info
> key value
--- -----
ip  172.17.1.1
ip  172.17.1.2
ip  172.17.1.3
> drop series from info where OS='CentOS'
> select * from info
> name: info
> time                OS      cpu_usage hostname ip         mem_usage sum_conn
> ----                --      --------- -------- --         --------- --------
> 1582622294550197570 Windows 45.54     OA       172.17.1.3 3.456     175
> 1582623263478460723 Windows 55.55     OA       172.17.1.3 5.555     55
> 1582623300598878112 Windows 22.55     OA       172.17.1.3 2.555     25
> exit
> [root@influx meta]# cd
> [root@influx ~]# cd backup/
> [root@influx backup]# tree mytest_all.bak/
> mytest_all.bak/
> ├── meta.00
> └── mytest.autogen.00003.00

0 directories, 2 files

[root@influx backup]# cd mytest_all.bak/
[root@influx mytest_all.bak]# influxd restore -metadir /var/lib/influxdb/meta/ .                                                                                        /meta.00
restore: backup path should be a valid directory: ./meta.00
[root@influx mytest_all.bak]# influxd restore -database mytest -datadir /var/lib                                                                                        /influxdb/data/ ./mytest.autogen.00003.00
restore: backup path should be a valid directory: ./mytest.autogen.00003.00
[root@influx mytest_all.bak]# influx
Connected to http://localhost:8086 version 1.7.10
InfluxDB shell version: 1.7.10
> show databases
> name: databases
> name
----
_internal
mytest
> use mytest
> Using database mytest

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
> select * from info
> name: info
> time                OS      cpu_usage hostname ip         mem_usage sum_conn
> ----                --      --------- -------- --         --------- --------
> 1582622294550197570 Windows 45.54     OA       172.17.1.3 3.456     175
> 1582623263478460723 Windows 55.55     OA       172.17.1.3 5.555     55
> 1582623300598878112 Windows 22.55     OA       172.17.1.3 2.555     25
> exit
> [root@influx mytest_all.bak]# cd
> [root@influx ~]# cd backup/
> [root@influx backup]# ll
> total 4
> drwx------ 2 root root 4096 Feb 27 13:43 mytest_all.bak
> [root@influx backup]# influxd restroe -database mytest -datadir /var/lib/d
> dbus/     dhclient/

[root@influx backup]# influxd restore -help

Uses backup copies from the specified PATH to restore databases or specific shards from InfluxDB OSS
  or InfluxDB Enterprise to an InfluxDB OSS instance.

Usage: influxd restore -portable [options] PATH

Note: Restore using the '-portable' option consumes files in an improved Enterprise-compatible
  format that includes a file manifest.

Options:
    -portable
            Required to activate the portable restore mode. If not specified, the legacy restore mode is used.
    -host  <host:port>
            InfluxDB OSS host to connect to where the data will be restored. Defaults to '127.0.0.1:8088'.
    -db    <name>
            Name of database to be restored from the backup (InfluxDB OSS or InfluxDB Enterprise)
    -newdb <name>
            Name of the InfluxDB OSS database into which the archived data will be imported on the target system.
            Optional. If not given, then the value of '-db <db_name>' is used.  The new database name must be unique
            to the target system.
    -rp    <name>
            Name of retention policy from the backup that will be restored. Optional.
            Requires that '-db <db_name>' is specified.
    -newrp <name>
            Name of the retention policy to be created on the target system. Optional. Requires that '-rp <rp_name>'
            is set. If not given, the '-rp <rp_name>' value is used.
    -shard <id>
            Identifier of the shard to be restored. Optional. If specified, then '-db <db_name>' and '-rp <rp_name>' are
            required.
    PATH
            Path to directory containing the backup files.

restore: flag: help requested

[root@influx backup]# ls
mytest_all.bak
[root@influx backup]# tree mytest_all.bak/
mytest_all.bak/
├── meta.00
└── mytest.autogen.00003.00

0 directories, 2 files
[root@influx backup]# influx
influx          influxd         influx_inspect  influx_stress   influx_tsm
[root@influx backup]# influx
Connected to http://localhost:8086 version 1.7.10
InfluxDB shell version: 1.7.10
> show databases
> name: databases
> name
----
_internal
mytest
> use mytest
> Using database mytest

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
> exit
> [root@influx backup]# ll
> total 4
> drwx------ 2 root root 4096 Feb 27 13:43 mytest_all.bak
> [root@influx backup]# influxd restore -portable -db mytest1 ./mytest_all.bak/
> restore: No manifest files found in: ./mytest_all.bak/


> [root@influx mytest_all.bak]# influxd backup -portable -database mytest /root/backup/
> 2020/02/27 14:38:04 backing up metastore to /root/backup/meta.00
> 2020/02/27 14:38:04 backing up db=mytest
> 2020/02/27 14:38:04 backing up db=mytest rp=autogen shard=3 to /root/backup/mytest.autogen.00003.00 since 0001-01-01T00:00:00Z
> 2020/02/27 14:38:04 backup complete:
> 2020/02/27 14:38:04     /root/backup/20200227T063804Z.meta
> 2020/02/27 14:38:04     /root/backup/20200227T063804Z.s3.tar.gz
> 2020/02/27 14:38:04     /root/backup/20200227T063804Z.manifest
> [root@influx mytest_all.bak]# ll
> total 12
> -rw-r--r-- 1 root root  265 Feb 27 13:43 meta.00
> -rw-r--r-- 1 root root 4608 Feb 27 13:43 mytest.autogen.00003.00
> [root@influx mytest_all.bak]# cd ..
> [root@influx backup]# ll
> total 16
> -rw------- 1 root root  293 Feb 27 14:38 20200227T063804Z.manifest
> -rw-r--r-- 1 root root  241 Feb 27 14:38 20200227T063804Z.meta
> -rw------- 1 root root 1036 Feb 27 14:38 20200227T063804Z.s3.tar.gz
> drwx------ 2 root root 4096 Feb 27 13:43 mytest_all.bak

> [root@influx mytest]# influxd restore -portable -db mytest -newdb mytest1 /root/backup/mytest
> 2020/02/27 14:45:07 Restoring shard 3 live from backup 20200227T063902Z.s3.tar.gz
> [root@influx mytest]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> show databases
> name: databases
> name
----
_internal
mytest
mytest1
> use mytest1
> Using database mytest1

# 备份出的数据是float类型，源表是int类型提示报错
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

> select *i INTO mytest..:MEASUREMENT from /.*/ GROUP BY *
> ERR: error parsing query: found i, expected FROM at line 1, char 9
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

> show measurements
> name: measurements
> name
----
info
test3
test4
test_1
test_cpu
> use mytest1
> Using database mytest1
> select * INTO mytest..:MEASUREMENT from /.*/ GROUP BY *
> name: result
> time written
---- -------
0    13
> use mytest
> Using database mytest
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
> select * test2
> ERR: error parsing query: found test2, expected FROM at line 1, char 10
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
> [root@influx mytest]# influxd restore -portable -db mytest -newdb mytest1 -rp autogen -newrp autogen_bak /root/backup/mytest
> 2020/02/27 14:56:58 error updating meta: DB metadata not changed. database may already exist
> restore: DB metadata not changed. database may already exist
> [root@influx mytest]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> use mytest
> Using database mytest
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
> show databases
> name: databases
> name
----
_internal
mytest
mytest1
> show retention policies on "mytest"
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 0s       168h0m0s           1        true
> show retention policies on mytest
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 0s       168h0m0s           1        true
> show retention policies on 'mytest'
> ERR: error parsing query: found mytest, expected identifier at line 1, char 27
> use mytest1
> Using database mytest1
> show retention policies on 'mytest1'
> ERR: error parsing query: found mytest1, expected identifier at line 1, char 27
> show retention policies on mytest1
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 0s       168h0m0s           1        true
> use mytest
> Using database mytest
> drop retention policy "autogen" on "mytest"
> show retention policies on mytest
> name duration shardGroupDuration replicaN default
---- -------- ------------------ -------- -------
> exit
> [root@influx mytest]# influxd restore -portable -db mytest -newdb mytest1 -rp autogen -newrp autogen_bak /root/backup/mytest
> 2020/02/27 15:04:04 error updating meta: DB metadata not changed. database may already exist
> restore: DB metadata not changed. database may already exist
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
> use mytest2
> Using database mytest2
> select * into mytest..:measurement from /mytest2.autogen_bak.*/ group by *
> name: result
> time written
---- -------
0    0
> select * into mytest..:measurement from /.*/ group by *
> ERR: retention policy not found: autogen
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
> select * into mytest.autogen_new.:measurement from /mytest2.autogen_bak.*/ group by *
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
> [root@influx mytest]# influxd restore -portable -db mytest -newdb mytest3 /root/backup/mytest
> 2020/02/27 15:11:55 Restoring shard 3 live from backup 20200227T063902Z.s3.tar.gz
> [root@influx mytest]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> use mytest3
> Using database mytest3
> show retention policies on mytest3
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 0s       168h0m0s           1        true
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
> exit
> [root@influx mytest]# influxd backup -portable -database mytest /root/backup/mytest_new
> 2020/02/27 15:13:27 backing up metastore to /root/backup/mytest_new/meta.00
> 2020/02/27 15:13:27 backing up db=mytest
> 2020/02/27 15:13:27 backup complete:
> 2020/02/27 15:13:27     /root/backup/mytest_new/20200227T071327Z.meta
> 2020/02/27 15:13:27     /root/backup/mytest_new/20200227T071327Z.manifest
> [root@influx mytest]# influxd restore -portable -db mytest -newdb mytest_4 /root/backup/mytest_new
> 2020/02/27 15:14:07 error updating meta: DB metadata not changed. database may already exist
> restore: DB metadata not changed. database may already exist
> [root@influx mytest]# influxd restore -portable -db mytest -newdb mytest8 /root/backup/mytest_new
> 2020/02/27 15:14:31 error updating meta: DB metadata not changed. database may already exist
> restore: DB metadata not changed. database may already exist
> [root@influx mytest]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> show databases
> name: databases
> name
----
_internal
mytest
mytest1
mytest2
mytest3
mytest_4
mytest8
> use mytest8
> Using database mytest8
> show measurements
> exit
> [root@influx mytest]# influxd restore -portable -db mytest -newdb mytestt /root/backup/mytest_new
> 2020/02/27 15:15:49 error updating meta: DB metadata not changed. database may already exist
> restore: DB metadata not changed. database may already exist
> [root@influx mytest]# ll
> total 12
> -rw------- 1 root root  293 Feb 27 14:39 20200227T063902Z.manifest
> -rw-r--r-- 1 root root  241 Feb 27 14:39 20200227T063902Z.meta
> -rw------- 1 root root 1036 Feb 27 14:39 20200227T063902Z.s3.tar.gz
> [root@influx mytest]# cd ..
> [root@influx backup]# ll
> total 24
> -rw------- 1 root root  293 Feb 27 14:38 20200227T063804Z.manifest
> -rw-r--r-- 1 root root  241 Feb 27 14:38 20200227T063804Z.meta
> -rw------- 1 root root 1036 Feb 27 14:38 20200227T063804Z.s3.tar.gz
> drwx------ 2 root root 4096 Feb 27 14:39 mytest
> drwx------ 2 root root 4096 Feb 27 13:43 mytest_all.bak
> drwx------ 2 root root 4096 Feb 27 15:13 mytest_new
> [root@influx backup]# rm -rf *
> [root@influx backup]# ll
> total 0
> [root@influx backup]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> show databases
> name: databases
> name
----
_internal
mytest
mytest1
mytest2
mytest3
mytest_4
mytest8
mytestt
> drop database mytestt
> drop database mytest8
> drop database mytest_4
> drop database mytest3
> drop database mytest2
> drop database mytest1
> show databases
> name: databases
> name
----
_internal
mytest
> use mytest
> Using database mytest
> show measurements
> show measurements
> show measurements
> drop database mytest
> create database mytest
> ackup/mytest_new
> ERR: error parsing query: found ackup, expected SELECT, DELETE, SHOW, CREATE, DROP, EXPLAIN, GRANT, REVOKE, ALTER, SET, KILL at line 1, char 1
> show measurements
>
> insert info,hostname=DB,OS=CentOS cpu_usage=60.11,mem_usage=6.453
> insert info,ip=172.17.1.2,hostname=DB,OS=CentOS cpu_usage=45.44,mem_usage=4.753
> insert info,ip=172.17.1.1,hostname=DB,OS=CentOS cpu_usage=60.33,mem_usage=6.453 1582619436829745192
> select * from info where OS='' or ip=''
> name: info
> time                OS     cpu_usage hostname ip mem_usage
> ----                --     --------- -------- -- ---------
> 1582788240252519327 CentOS 60.11     DB          6.453
> delete from info where OS='' or ip=''
> insert info,ip=172.17.1.3,hostname=OA,OS=Windows cpu_usage=45.54,mem_usage=3.456,sum_conn=175i
> select * from info
> name: info
> time                OS      cpu_usage hostname ip         mem_usage sum_conn
> ----                --      --------- -------- --         --------- --------
> 1582619436829745192 CentOS  60.33     DB       172.17.1.1 6.453
> 1582788256292315622 CentOS  45.44     DB       172.17.1.2 4.753
> 1582788334812734722 Windows 45.54     OA       172.17.1.3 3.456     175
> insert info,ip=172.17.1.1,OS=CentOS,hostname=DB sum_conn=99i 1582619436829745192
> insert info,ip=172.17.1.2,OS=CentOS,hostname=DB sum_conn=88i 1582788256292315622


> select * from info
> name: info
> time                OS      cpu_usage hostname ip         mem_usage sum_conn
> ----                --      --------- -------- --         --------- --------
> 1582619436829745192 CentOS  60.33     DB       172.17.1.1 6.453     99
> 1582788256292315622 CentOS  45.44     DB       172.17.1.2 4.753     88
> 1582788334812734722 Windows 45.54     OA       172.17.1.3 3.456     175
>
> 
>
> insert info,ip=172.17.1.1,hostname=DB,OS=CentOS cpu_usage=77.77,mem_usage=7.777,sum_conn=77i
> insert info,ip=172.17.1.2,hostname=DB,OS=CentOS cpu_usage=66.66,mem_usage=6.666,sum_conn=66i
> insert info,ip=172.17.1.3,hostname=OA,OS=Windows cpu_usage=55.55,mem_usage=5.555,sum_conn=55i
> insert info,ip=172.17.1.1,hostname=DB,OS=CentOS cpu_usage=44.77,mem_usage=4.777,sum_conn=47i
> insert info,ip=172.17.1.2,hostname=DB,OS=CentOS cpu_usage=33.66,mem_usage=3.666,sum_conn=36i
> insert info,ip=172.17.1.3,hostname=OA,OS=Windows cpu_usage=22.55,mem_usage=2.555,sum_conn=25i
>
> 
>
> select * from inf
> ERR: error parsing query: found INF, expected identifier at line 1, char 15
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
> insert info,ip=172.17.1.3,hostname=OA,OS=Windows cpu_usage=22.55,mem_usage=2.555,sum_conn=25
> ERR: {"error":"partial write: field type conflict: input field \"sum_conn\" on measurement \"info\" is type float, already exists as type integer dropped=1"}

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
> [root@influx backup]# ll
> total 0
> [root@influx backup]# influxd backup -portable -database mytest /root/backup/mytest
> 2020/02/27 15:36:13 backing up metastore to /root/backup/mytest/meta.00
> 2020/02/27 15:36:13 backing up db=mytest
> 2020/02/27 15:36:13 backing up db=mytest rp=autogen shard=8 to /root/backup/mytest/mytest.autogen.00008.00 since 0001-01-01T00:00:00Z
> 2020/02/27 15:36:13 backup complete:
> 2020/02/27 15:36:13     /root/backup/mytest/20200227T073613Z.meta
> 2020/02/27 15:36:13     /root/backup/mytest/20200227T073613Z.s8.tar.gz
> 2020/02/27 15:36:13     /root/backup/mytest/20200227T073613Z.manifest
> [root@influx backup]# ll
> total 4
> drwx------ 2 root root 4096 Feb 27 15:36 mytest
> [root@influx backup]# cd mytest/
> [root@influx mytest]# ll
> total 12
> -rw------- 1 root root 293 Feb 27 15:36 20200227T073613Z.manifest
> -rw-r--r-- 1 root root 241 Feb 27 15:36 20200227T073613Z.meta
> -rw------- 1 root root 923 Feb 27 15:36 20200227T073613Z.s8.tar.gz
> [root@influx mytest]# cd ..

> [root@influx backup]# influxd restore -portable -database mytest -newdb mytest1 /root/backup/mytest
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
> use myest1
> ERR: Database myest1 doesn't exist. Run SHOW DATABASES for a list of existing databases.
> DB does not exist!
> use mytest1
> Using database mytest1
> select * into mytest..:measurement from /.*/ group by *
> ERR: retention policy not found: autogen
> exit
> [root@influx backup]# influxd restore -portable -database mytest -rp autogen /root/backup/mytest
> 2020/02/27 15:43:36 error updating meta: DB metadata not changed. database may already exist
> restore: DB metadata not changed. database may already exist
> [root@influx backup]# influxd restore -portable -database mytest -newdb mytest1 -rp autogen -newrp autogen_bak /root/backup/mytest
> 2020/02/27 15:44:17 error updating meta: DB metadata not changed. database may already exist
> restore: DB metadata not changed. database may already exist
> [root@influx backup]# influxd restore -portable -database mytest -newdb mytest2 -rp autogen -newrp autogen_bak /root/backup/mytest
> 2020/02/27 15:44:32 Restoring shard 8 live from backup 20200227T073613Z.s8.tar.gz
> [root@influx backup]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> use mytest2
> Using database mytest2
> show measurements
> name: measurements
> name
----
info
test2
test3
> use mytest
> Using database mytest
> select * into mytest..:measurement from /.*/ group by *
> ERR: retention policy not found: autogen
> create retention policy "autogen" on "mytest" duration 3w replication 1 default
> select * into mytest..:measurement from /.*/ group by *
> name: result
> time written
---- -------
0    0
> use mytest
> Using database mytest
> show measurements
> use mytest2
> Using database mytest2
> select * into mytest..:measurement from /.*/ group by *
> name: result
> time written
---- -------
0    11
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
> show field from test2
> ERR: error parsing query: found FROM, expected KEY, KEYS at line 1, char 12
> show field kyes from test2
> ERR: error parsing query: found kyes, expected KEY, KEYS at line 1, char 12
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
> show retention policies on "mytest"
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 504h0m0s 24h0m0s            1        true
> select * into mytest.autogen.:measurement from /mytest2.autogen_bak.*/ group by *
> name: result
> time written
---- -------
0    0
> show retention policies on "mytest"
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 504h0m0s 24h0m0s            1        true
> use mytest2
> Using database mytest2
> show retention policies on "mytest2"
> name        duration shardGroupDuration replicaN default
> ----        -------- ------------------ -------- -------
> autogen_bak 0s       168h0m0s           1        true
> show retention policies
> name        duration shardGroupDuration replicaN default
> ----        -------- ------------------ -------- -------
> autogen_bak 0s       168h0m0s           1        true
> exit
> [root@influx backup]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> show retention policies
> ERR: database name required
> Warning: It is possible this error is due to not setting a database.
> Please set a database with the command "use <database>".
> use mytest
> Using database mytest
> show retention policies
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 504h0m0s 24h0m0s            1        true
> alter database mytest rename mytest4
> ERR: error parsing query: found DATABASE, expected RETENTION at line 1, char 7
> exit
> [root@influx backup]# influxd backup -portable -database mytest /root/backup/mytestwww
> 2020/02/27 15:57:01 backing up metastore to /root/backup/mytestwww/meta.00
> 2020/02/27 15:57:01 backing up db=mytest
> 2020/02/27 15:57:01 backing up db=mytest rp=autogen shard=11 to /root/backup/mytestwww/mytest.autogen.00011.00 since 0001-01-01T00:00:00Z
> 2020/02/27 15:57:01 backing up db=mytest rp=autogen shard=12 to /root/backup/mytestwww/mytest.autogen.00012.00 since 0001-01-01T00:00:00Z
> 2020/02/27 15:57:01 backup complete:
> 2020/02/27 15:57:01     /root/backup/mytestwww/20200227T075701Z.meta
> 2020/02/27 15:57:01     /root/backup/mytestwww/20200227T075701Z.s11.tar.gz
> 2020/02/27 15:57:01     /root/backup/mytestwww/20200227T075701Z.s12.tar.gz
> 2020/02/27 15:57:01     /root/backup/mytestwww/20200227T075701Z.manifest
> [root@influx backup]# influxd restore -portable -database mytest -newdb mytest4 -rp autogen -newrp autogen_bak /root/backup/mytestwww
> 2020/02/27 15:57:29 Restoring shard 11 live from backup 20200227T075701Z.s11.tar.gz
> 2020/02/27 15:57:29 Restoring shard 12 live from backup 20200227T075701Z.s12.tar.gz
> [root@influx backup]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> use mytest4
> Using database mytest4
> show retention policies
> name        duration shardGroupDuration replicaN default
> ----        -------- ------------------ -------- -------
> autogen_bak 504h0m0s 24h0m0s            1        true
> select * into mytest2.autogen_bak.:measurement from /mytest4.autogen_bak.*/ group by *
> name: result
> time written
---- -------
0    0
> show retention policies on mytest2
> name        duration shardGroupDuration replicaN default
> ----        -------- ------------------ -------- -------
> autogen_bak 0s       168h0m0s           1        true
> select * into mytest.autogen.:measurement from /mytest4.autogen_bak.*/ group by *
> name: result
> time written
---- -------
0    0
> show retention policies on mytest
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 504h0m0s 24h0m0s            1        true
> exit
> [root@influx backup]# influxd restore -portable -database mytest -rp autogen -newrp autogen_bak /root/backup/mytestwww
> 2020/02/27 16:01:52 error updating meta: DB metadata not changed. database may already exist
> restore: DB metadata not changed. database may already exist
> [root@influx backup]# influxd restore -portable -database mytest2 -rp autogen -newrp autogen_bak /root/backup/mytestwww
> 2020/02/27 16:02:18 error updating meta: DB metadata not changed. database may already exist
> restore: DB metadata not changed. database may already exist
> [root@influx backup]#
> [root@influx backup]#
> [root@influx backup]#
> [root@influx backup]# influx
> Connected to http://localhost:8086 version 1.7.10
> InfluxDB shell version: 1.7.10
> show retention policies on mytest
> name    duration shardGroupDuration replicaN default
> ----    -------- ------------------ -------- -------
> autogen 504h0m0s 24h0m0s            1        true
> show retention policies on mytest2
> name        duration shardGroupDuration replicaN default
> ----        -------- ------------------ -------- -------
> autogen_bak 0s       168h0m0s           1        true
> show retention policies on mytest3
> ERR: database not found: mytest3
> Warning: It is possible this error is due to not setting a database.
> Please set a database with the command "use <database>".
> show retention policies on mytest3
> ERR: database not found: mytest3
> Warning: It is possible this error is due to not setting a database.
> Please set a database with the command "use <database>".
> show retention policies on mytest4
> name        duration shardGroupDuration replicaN default
> ----        -------- ------------------ -------- -------
> autogen_bak 504h0m0s 24h0m0s            1        true
> select * into mytest.autogen_bak.:measurement from /mytest4.autogen_bak.*/ group by *
> ERR: database name required
> Warning: It is possible this error is due to not setting a database.
> Please set a database with the command "use <database>".
> use mytest
> Using database mytest
> select * into mytest.autogen_bak.:measurement from /mytest4.autogen_bak.*/ group by *
> name: result
> time written
---- -------
0    0
> use mytest4
> Using database mytest4
> select * into mytest.autogen_bak.:measurement from /mytest4.autogen_bak.*/ group by *
> name: result
> time written
---- -------
0    0
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
