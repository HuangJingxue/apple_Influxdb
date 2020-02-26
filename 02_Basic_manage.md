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
