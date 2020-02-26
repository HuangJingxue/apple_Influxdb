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

```
