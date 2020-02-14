
### 时序数据库学习

- [入门](https://docs.influxdata.com/influxdb/v1.7/)
- [备份和恢复](https://docs.influxdata.com/influxdb/v1.7/administration/backup_and_restore/)
```wiki
重点提示：
[ -portable ]：以较新的InfluxDB Enterprise兼容格式生成备份文件。强烈建议所有InfluxDB OSS用户使用。
如果-portable未指定，则使用默认的旧式备份实用程序-除非-database指定，否则仅备份主机元存储。如果不使用-portable，请查看下面的“ 备份（旧版）”以了解预期的行为。
InfluxDB元存储包含有关系统状态的内部信息，包括用户信息，数据库和分片元数据，连续查询，保留策略和订阅。在节点运行时，可以通过运行以下命令来创建实例的元存储的备份：
```
- [influxQL与SQL](https://docs.influxdata.com/influxdb/v1.7/concepts/crosswalk/)

+ influxdb解决什么问题？
```wiki
常用的一种使用场景：监控数据统计。
InfluxDB是一个由InfluxData开发的开源时序型数据。它由Go写成，着力于高性能地查询与存储时序型数据。InfluxDB被广泛应用于存储系统的监控数据，IoT行业的实时数据等场景。
```
+ 我们如何使用influxdb？
```wiki
  1.安装
  2.备份恢复
  3.内存高如何解决？
  4.数据如何存放？
  5.怎么查看内容？
```
#### 怎么查看内容


+ 如何跟生态产品结合使用？
