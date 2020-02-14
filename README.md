
### 时序数据库学习
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
|概念|MySQL|InfluxDB|
|数据库（同）|	database	|database|
|表（不同）	|table|	measurement|
|列（不同）	|column|	tag(带索引的，非必须)、field(不带索引)、timestemp(唯一主键)|

+ 如何跟生态产品结合使用？
