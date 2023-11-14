---
date: 2023-11-13 16:29:02
linktitle: influxdb介绍

title: influxdb介绍
weight: 10
tags : [
    "hugo",
]

---

## Step 1. 准备条件

- git下载
- hugo下载



- go写的
- 官方有OSS单机开源版(官方不支持集群) 。
- InfluxDB 1.8.x稳定版本
- InfluxDB 2.0 正式版本，该版本又分为两个系列，云模式的 InfluxDB Cloud 和独立部署的 InfluxDB OSS
- ` `集群版闭源，走商业路线
- 主流版本有1.X 和 2.X 1.X以1.8为主流（https://developer.aliyun.com/article/1265960）
- 2.x的区别
- 1整合了TICK 四个组件，分别对应Telegraf数据采集，InfluxDB数据存储，Chronograf WebUI数据可视化，Kapacitor 告警
- 权限增强，支持token读写数据
- 增加查询语言 Flux语法

- 场景：适合 海量时许数据的**插入**和**查询**的场景（ 注意：删除和更新 支持很少）
- 数据分析，[IoT](https://so.csdn.net/so/search?q=IoT&spm=1001.2101.3001.7020)设备数据采集，监控告警
- 性能参考  
- influxdb版本 1.8.10
- 本身机器硬件参数（32G内存  16核CPU标准版）
- ` `数据量、网络带宽
- mearsurement个数、field个数、tag个数
- 举例来说，基本的服务器配置 32G内存  16核，支持每s几W的写入 没什么问题，正常的查询在1-20ms之内

|Plain Text<br><br>网上测试参考<br>`    `8G内存 2核cpu <br>`    `最大的吞吐量为每秒写入60万条数据（3个字段，1个索引）<br>`    `读取性能 2ms/条<br><br>环卫云项目参考<br>`    `32G   16核cpu  1000Mbps的带宽<br>`    `平均吃  8G内存   600% cpu<br>`    `插入 每秒写入  2-3K条数据（50个字段、2-3个索引）|
| :- |
- retention policy：数据存储策略（默认策略为autogen）InfluxDB没有删除数据操作，规定数据的保留时间达到清除数据的目的；
- 底层时间戳为纳秒， 存储用的是utc时间戳（自1970年1月1日起的相对时间戳）
- 支持通过  tz（Asia/China）转化成对应时区的时间
- https://jasper-zhang1.gitbooks.io/influxdb/content/Guide/writing\_data.html
- 功能性
- HTTP协议的API 
- 通过数据库本身的聚合函数
- 横向对比
- 在时序数据库出现之前，存储时序数据这项领域一直被 MongoDB 所占据，在数据量不够高的情况下，InfluxDB 并不会比 MongoDB 或者 ElasticSearch 有更明显的优势
- MySQL、Oracle 等，优点是具有 ACID 的特性，各方面能力都比较均衡。缺点是查询、插入和修改的性能都很一般。
- 基于 PostgreSQL 的 TimeScaleDB
- 高性能写入、高性能查询、低存储成本、支持海量时间线、弹性
- 纵向
- 数据库名、存储策略、 measurement （类似 mysql 的表）名与 tag 名一起作为时间轴的标记 (series)，series是存在内存中的，所以不要太多series，影响性能

**2 部署安装**

https://gitee.com/ming-liang-zhou/influxdb/blob/master/influxdb.md

默认配置

|Java<br>默认influxDB使用以下端口<br>8086: 用于客户端和服务端交互的HTTP API<br>8088: 用于提供备份和恢复的RPC服务<br><br>配置文件path <br>` `/etc/influxdb/influxdb.conf<br><br>密码<br>` `默认用户名、密码是admin、password<br> <br> <br>` `#influx 进入influx cmd<br>` `influx<br>` `#添加一个管理员用户<br>` `create user "root" with password 'root' with all privileges<br><br>默认时间戳<br>`   `时间戳默认用的utc, db底层存储的单位是纳秒（等值查询时） 1685943000000000 <br>`   `influxDB使用所在主机的本地时间的UTC时间(比国内晚8个小时)来设置timestamp|
| :- |
QA:

`    `1.chronograf的端口8888被占用，修改配置文件后重启

|Java<br><br>经过测试后，需要修改以下文件中的端口配置才能修改默认端口<br>/usr/lib/systemd/system/chronograf.service<br>/usr/lib/chronograf/scripts 中的chronograf.service 和init.sh文件中的端口<br><br>执行更新命令<br>systemctl daemon-reload<br>重新启动chronograf后可以正常打开<br>sudo systemctl start chronograf|
| :- |
**3 Cmd**

|Java<br># service方式<br># 启动命令 service influxdb start<br># 重启命令 service influxdb restart<br># 关闭命令 service influxdb stop<br># 查看启动日志 sudo journalctl -u influxdb.service -f<br><br># systemctl 方式<br># 启动命令 sudo systemctl start influxdb<br># 关闭命令 sudo systemctl stop  influxdb<br><br><br><br>#查看influxdb进程<br>ps -ef|grep influxdb<br><br>#杀进程<br>kill -9 进程id<br><br>#其他参考<br>#启动influxdb<br>nohup /mnt/app/mydata/influxdb/influxdb-1.8.10-1/usr/bin/influxd -config /mnt/app/mydata/influxdb/influxdb-1.8.10-1/etc/influxdb/influxdb.conf >>/mnt/app/mydata/influxdb/influxdb-1.8.10-1/var/log/influxdb/nohup.log 2>&1 &<br><br><br>#可视化面板相关<br>#查看chronograf进程<br>ps -ef|grep chronograf<br>#杀进程<br>kill -9 进程id<br>#启动chronograf<br>nohup /mnt/app/mydata/influxdb/chronograf-1.9.4-1/usr/bin/chronograf >> /mnt/app/mydata/influxdb/chronograf-1.9.4-1/var/log/chronograf/nohup.log 2>&1 &<br><br>nohup /tmp/zjf/chronograf-1.9.4-1/usr/bin/chronograf >> /tmp/zjf/chronograf-1.9.4-1/var/log/chronograf/nohup.log 2>&1 &|
| :- |



##https://www.metabase.com/docs/latest/installation-and-operation/running-metabase-on-docker

http://172.16.112.9:3000/  cowa2022..


|Rust<br>./influx -ssl -unsafeSsl -username iot -password cowa\_iot\_influx<br><br><br>## https://docs.influxdata.com/influxdb/v1.8/<br>## https://jasper-zhang1.gitbooks.io/influxdb/content/<br>1.创建数据库iot<br>create database iot;<br>2.使用数据库<br>use iot;<br>3.查询所有measurement<br>show measurements;<br>4.创建管理员用户admin<br>create user "admin" with password '密码' with all privileges;<br>5.创建普通用户iot<br>create user "iot" with password '密码';<br>6.创建普通用户iotRead<br>create user "iotRead" with password '密码';<br>7.给iot用户授权iot数据库只写权限<br>grant write on "iot" to "iot";<br>8.给iotRead用户授权iot数据库只读权限<br>grant read on "iot" to "iotRead";<br>9.创建默认存储策略（rp\_30d），数据保留近30天<br>create retention policy "rp\_30d" on "iot" duration 30d replication 1 default;<br>10.创建存储策略（rp\_10d），数据保留近10天<br>create retention policy "rp\_10d" on "iot" duration 10d replication 1;<br>11.创建存储策略（rp\_3d），数据保留近3天<br>create retention policy "rp\_3d" on "telegraf" duration 3d replication 1;<br>12.连续查询<br>#mydb 中新建了一个名为 cq\_30m 的连续查询，每三十分钟取一个used字段的平均值，加入 mem\_used\_30m 表中<br>create continuous query cq\_30m on mydb begin select mean(used) into men\_used\_30s from men group by time(30m) end;<br># 查看所有的连续查询<br>show continuous queries;<br># 删除mydb 数据库中的连续查询 cq\_30m<br>drop continuous query cq\_30m  on  mydb<br>13.常用函数<br>#count()函数 返回一个（field）字段中的非空值的数量<br>select count(\*) from disk\_free<br><br>#DISTINCT()函数 返回一个字段（field）的唯一值，函数里面是 field 值！<br>select distinct(value) from disk\_free<br><br>#MEAN() 函数 返回一个字段（field）中的值的算术平均值（平均值）字段类型必须是长整型或float64<br>select mean(value) from disk\_free<br><br>#MEDIAN()函数 从单个字段（field）中的排序值返回中间值（中位数）<br>#字段值的类型必须是长整型或float64格式<br>select median(value) from disk\_free<br><br>#SPREAD()函数 返回字段的最小值和最大值之间的差值<br>select spread(value) from disk\_free<br><br>#SUM()函数 返回一个字段中的所有值的和<br>select sum(value) from disk\_free<br><br>#TOP()函数 返回一个字段中最大的N个值<br>select top("value", 1) from disk\_free<br><br>#BOTTOM()函数 返回一个字段中最小的N个值<br>select bottom("value",2) from disk\_free<br><br>#FIRST()函数 返回一个字段中存活时间最长的值<br>select first(value) from disk\_free<br><br>#MAX()函数 返回一个字段中的最大值<br>#MIN()函数 返回一个字段中的最小值<br>select max(value) from disk\_free<br>select min(value) from disk\_free<br><br>#PERCENTILE()函数 返回排序值排位为N的百分值<br>#value 字段按照不同的 value求百分比，然后取第五位数据。<br>select percentile(value,99),value from disk\_free<br><br>PERCENTILE(<field\_key>,100)相当于MAX(<field\_key>)。<br>PERCENTILE(<field\_key>, 50)几乎等同于MEDIAN(<field\_key>)，但MEDIAN()如果字段键包含偶数个字段值，则该函数返回两个中间值的平均值。<br>PERCENTILE(<field\_key>,0)不等于MIN(<field\_key>)。<br><br>FAQ：<br>1.一个点由measurement名称，tag set和timestamp唯一标识。如果您提交具有相同measurement，tag set和timestamp，但具有不同field set的行协议，则field set将变为旧field set与新field set的合并，并且如果有任何冲突以新field set为准。|
| :- |
**4 sql操作**

|Java<br>插入<br>`    `tag 字段插入后不能删除，慎重！<br>`    `field字段插入后不能删除，慎重！<br>    <br>    <br>delete<br>`    `influxdb作为时序数据库的一种，是不提供类似mysql的delete操作，也就是，原则上，你删不了数据，除非删库。 但是，influxdb提供了数据保留策略policies，通过对policies操作，可以达到删除数据的效果(1)。可以根据time和tag进行批量删除，不支持基于field进行批量删除，只能一条一条删除(2)。DELETE SQL删除数据的过程是：根据DELETE SQL中指定的WHERE条件，从tsi/tsl文件中过滤得到满足条件的seriesID，通过查找series index以及series segment文件得到seriesID对应的seriesKey(3)。|
| :- |



**基础数据**

|Java<br>**查询数据库大小**<br>select sum(diskBytes) / 1024 / 1024 /1024 from \_internal."monitor"."shard" where time > now() - 10s<br>select sum(diskBytes) / 1024 / 1024 /1024 from \_internal."monitor"."shard" where time > now() - 10s group by "database"<br><br>**查询记录条数**<br>SELECT count(\*) FROM "dataAnalysis"."autogen"."carGis" <br>SELECT count(\*) FROM "dataAnalysis"."autogen"."carGis" where time>'2022-09-22 12:00:00'  and  time<'2022-09-23 00:00:00'  tz('Asia/Shanghai')     中国时区---东八<br>SELECT count(\*) FROM "dataAnalysis"."autogen"."carGis" where time>'2022-09-22 12:00:00'  and  time<'2022-09-23 00:00:00'                                    UTC时区---东零<br><br><br>**查看数据库的表字段类型**<br>`  `show  field keys from dataAnalysis<br><br>**设置当前策略为default策略**<br>ALTER RETENTION POLICY "rp\_30d" ON "iot" DEFAULT|
| :- |
**query**

- time特殊                 不需要''或者""包起来   直接time
- tag和fields              需要""包起来   

`         `"tagkey"='30.22'   等号右边，若为字符串需要写成单引号

`         `如果写错了，则查询会没有结果

- 默认不写 tz('Asia/Shanghai')  表示查询的时间戳为 UTC，通常需要在最后加 tz('Asia/Shanghai')

|Java<br>查询部分字段<br>SELECT "latitude", "longitude" FROM "iot"."rp\_30d"."drive" WHERE time > :dashboardTime: AND time < :upperDashboardTime: AND "carNo"='80001' AND "longitude"='NaN'<br><br>查询全部字段<br>SELECT \* FROM "dataAnalysis"."autogen"."carGis" where time>'2022-09-22 00:00:00'  and  time<'2022-09-23 00:00:00'  tz('Asia/Shanghai') <br><br>时区<br>utc查询和 tz时区查询<br>` `time>'2022-09-22 00:00:00'  and  time<'2022-09-23 00:00:00'  <br>**等价于**<br>` `time>'2022-09-22 08:00:00'  and  time<'2022-09-23 08:00:00'  tz('Asia/Shanghai')|
| :- |
分页查询

**Insert**

- measurement加tags组成了一个series 
- tag的key和value都必须是字符串。
- ` `fields的key也是必须的，而且是字符串，默认情况下field的value是float类型的。
- ` `tag+timestamp的组合--->唯一定义1条记录，即相同time相同tag的唯一记录 

重复插入后面的会默认覆盖之前的记录

2  不支持update 重新插入相同记录来达到修改的效果

|Java<br>USE "zml\_test"."rp\_30d"; insert "zml\_test"."rp\_30d"."aaaaa"  "carno"='123' "bbb"='111' <br><br>USE "zml\_test"."rp\_30d"; insert "zml\_test"."rp\_30d"."zmlzzz"  "dev\_id"='1' "aa"='22' "bb"='111' time=1690767675736|
| :- |

**delete**

- 不支持删除tag或者field
- 通过retency来实现数据删除
- ` `表存储策略上：默认autogen无法删除数据  只能insert覆盖
- 非autogen策略表可以删除
- 支持删除time段的数据



|Plain Text<br>**1 根据时间点  精确到ns**<br>`    `delete from "dataAnalysis"."rp\_7d"."carGisT" where time=1688612365000<br>**2 根据时间范围**<br>`    `delete from "dataAnalysis"."rp\_7d"."carGisT" where time='2022-10-19 17:31:43.762'  tz('Asia/Shanghai') <br>`    `DELETE FROM <measurement\_name> WHERE [<tag\_key>='<tag\_value>'] | [<time interval>]<br><br>3 **删除某表的指定时间范围的数据**<br>`    `delete from  "dataAnalysis"."rp\_7d"."carGisT" where time<'2022-10-19 17:31:43.762Z' <br>`    `And time<'2022-10-19 17:31:43.762Z  tz('Asia/Shanghai') <br><br>4 **删除某个tag的所有记录**<br>`    `USE "zml\_test"."rp\_30d"; DROP SERIES FROM "mbb" WHERE "ccc" = '111'<br><br>5 **删除某个tag的时间范围内的记录(UTC)**<br>`    `USE  "iot"; DELETE FROM "drive" WHERE "carNo" = '1122333222' AND time > '2022-10-19 07:40:23'   AND time < '2022-10-19 19:40:23'   <br>`    `USE  "dataAnalysis"; DELETE FROM "carGisT" WHERE "carNo" = '1122333222' AND time= '2022-10-19 07:40:23'<br>`    `hds<br><br>6 **删除整表**<br>`    `USE "recloud\_test";  drop  measurement xxx  删除整表<br>`    `USE "recloud\_test";  delete from xxx   删除的是recloud\_test下所有策略的measurmentx   <br>`    `USE "iot";  delete from car\_detail|
| :- |

`   `80017                                                                19s   3s

1. beginTime: "2022-10-11 00:00:00"
2. carNo: "80017"
3. endTime: "2022-10-12 00:00:00"

SELECT \* FROM "iot"."rp\_30d"."drive" WHERE time > '2022-10-11T00:00:00Z' AND time < '2022-10-12T00:00:00Z' AND "carNo"='80017'

SELECT \* FROM "iot"."rp\_30d"."drive" WHERE time > '2022-10-11 00:00:00' AND time < '2022-10-12 00:00:00' AND "carNo"='80017'

SELECT count(\*) FROM "iot"."rp\_30d"."drive" where time > '2022-10-11T00:00:00Z' AND time < '2022-10-11T00:00:01Z' AND "carNo"='80017'

时差问题

chronograf查询----勾选local-----时间要减少8h

对应mysql   SELECT count(\*) FROM "dataAnalysis"."autogen"."carGis" 

SELECT count(\*) FROM "dataAnalysis"."autogen"."carGis" where time>'2022-09-22 12:00:00'  and  time<'2022-09-23 00:00:00'  tz('Asia/Shanghai')

联表查询

|Rust<br><br>SELECT "height", "latitude" FROM "iot"."rp\_30d"."drive" WHERE time > :dashboardTime: AND time < :upperDashboardTime: AND "carNo"='14001'<br><br><br>SELECT \* FROM "iot"."rp\_30d"."drive"，"iot"."rp\_30d"."gns" WHERE time > :dashboardTime: AND time < :upperDashboardTime: AND "carNo"='14001'|
| :- |

查询某段时间范围内的距今最近的1条

|Rust<br>select \* from recloud\_test.rp\_30d.iot\_carinfo\_flat where time > '2023-06-26 20:00:20' AND time < '2023-06-26 20:00:21' And \"carNo\"='14003' ORDER BY time Desc LIMIT 1 tz('Asia/Shanghai') <br><br><br>select \* from recloud\_test.rp\_30d.car\_detail where time > '2023-06-25 20:00:20' AND time < '2023-06-26 23:00:21'  ORDER BY time Desc LIMIT 1 tz('Asia/Shanghai') |
| :- |

查询 某字段不为空

| Rust<br>select \* from recloud\_test.rp\_30d.iot\_carinfo\_flat where And \"carNo\"='14003' ORDER BY time Desc LIMIT 1 tz('Asia/Shanghai') |
| :----------------------------------------------------------- |



