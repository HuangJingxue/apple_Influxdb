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
