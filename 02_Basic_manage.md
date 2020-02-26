#别名登录
```shell
[root@Oracle01 ~]# vim .bashrc 
alias qinxi_influx='influx -username admin -password Admin123 -precision 'rfc3339''

[root@Oracle01 ~]# source .bashrc
[root@Oracle01 ~]# qinxi_influx
Connected to http://localhost:8086 version 1.7.8
InfluxDB shell version: 1.7.8
> 
```

#插入数据
```sql
insert internal,host=A,eth=a v=1
insert internal,host=A,eth=a v=2
insert internal,host=A,eth=a v=3
insert internal,host=A,eth=a v=4
insert internal,host=A,eth=b v=2
insert internal,host=A,eth=b v=2
insert internal,host=A,eth=b v=2
insert internal,host=A,eth=b v=2
insert internal,host=B,eth=a v=1
insert internal,host=B,eth=a v=2
insert internal,host=B,eth=a v=3
insert internal,host=B,eth=a v=4
insert internal,host=B,eth=b v=2
insert internal,host=B,eth=b v=4
insert internal,host=B,eth=b v=1
insert internal,host=B,eth=b v=8

> show measurements
name: measurements
name
----
internal

> show tag keys
name: internal
tagKey
------
eth
host


> show field keys
name: internal
fieldKey fieldType
-------- ---------
v        float

> show series from internal
key
---
internal,eth=a,host=A
internal,eth=a,host=B
internal,eth=b,host=A
internal,eth=b,host=B


> select * from internal;
name: internal
time                           eth host v
----                           --- ---- -
2020-02-26T09:54:04.554629199Z a   A    1
2020-02-26T09:54:04.602011086Z a   A    2
2020-02-26T09:54:04.607964503Z a   A    3
2020-02-26T09:54:04.611256111Z a   A    4
2020-02-26T09:54:04.615987618Z b   A    2
2020-02-26T09:54:04.62674678Z  b   A    2
2020-02-26T09:54:04.630819322Z b   A    2
2020-02-26T09:54:04.635173729Z b   A    2
2020-02-26T09:54:04.641564847Z a   B    1
2020-02-26T09:54:04.645624095Z a   B    2
2020-02-26T09:54:04.653907878Z a   B    3
2020-02-26T09:54:04.659465839Z a   B    4
2020-02-26T09:54:04.662730791Z b   B    2
2020-02-26T09:54:04.669842722Z b   B    4
2020-02-26T09:54:04.676888859Z b   B    1
2020-02-26T09:54:05.962821609Z b   B    8

```

参考学习链接：https://www.jianshu.com/p/721e4ce4c066
```sql
SHOW FIELD KEYS --查看当前数据库所有表的字段
SHOW series from pay --查看key数据
SHOW TAG KEYS FROM "pay" --查看key中tag key值
SHOW TAG VALUES FROM "pay" WITH KEY = "merId" --查看key中tag 指定key值对应的值
SHOW TAG VALUES FROM cpu WITH KEY IN ("region", "host") WHERE service = 'redis'
DROP SERIES FROM <measurement_name[,measurement_name]> WHERE <tag_key>='<tag_value>' --删除key
SHOW CONTINUOUS QUERIES   --查看连续执行命令
SHOW QUERIES  --查看最后执行命令
KILL QUERY <qid> --结束命令
SHOW RETENTION POLICIES ON mydb  --查看保留数据
查询数据
SELECT * FROM /.*/ LIMIT 1  --查询当前数据库下所有表的第一行记录
select * from pay  order by time desc limit 2
select * from  db_name."POLICIES name".measurement_name --指定查询数据库下数据保留中的表数据 POLICIES name数据保留
删除数据
delete from "query" --删除表所有数据，则表就不存在了
drop MEASUREMENT "query"   --删除表（注意会把数据保留删除使用delete不会）
DELETE FROM cpu
DELETE FROM cpu WHERE time < '2000-01-01T00:00:00Z'
DELETE WHERE time < '2000-01-01T00:00:00Z'
DROP DATABASE “testDB” --删除数据库
DROP RETENTION POLICY "dbbak" ON mydb --删除保留数据为dbbak数据
DROP SERIES from pay where tag_key='' --删除key中的tag

SHOW SHARDS  --查看数据存储文件
DROP SHARD 1
SHOW SHARD GROUPS
SHOW SUBSCRIPTIONS
```
