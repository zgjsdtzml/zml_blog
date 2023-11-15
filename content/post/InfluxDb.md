---

---

## 1. 介绍

- git下载、hugo下载



- go写的，场景：适合 海量时许数据的**插入**和**查询**的场景（ 注意：删除和更新 支持很少）

  - 监控告警
  - iot物联网设备数据采集

- 官方有OSS单机开源版、官方集群版闭源收费，走商业路线。

  - 主流版本有1.X 和 2.X 1.X以[1.8为主流](https://developer.aliyun.com/article/1265960)

  - InfluxDB 2.0 正式版本，该版本又分为两个系列，云模式的 InfluxDB Cloud 和独立部署的 InfluxDB OSS

  - 2.x的区别

    - 1整合了TICK 四个组件，分别对应Telegraf数据采集，InfluxDB数据存储，Chronograf WebUI数据可视化，Kapacitor 告警

    - 权限增强，支持token读写数据

    - 增加查询语言 Flux语法

      

- 性能

  - 影响因素
    - 本身机器硬件参数（32G内存  16核CPU标准版）
    - 数据量、网络带宽
    - influxdb版本 1.8.10、mearsurement个数、field个数、tag个数

- **举例来说，基本的服务器配置 32G内存  16核，支持每s几W的写入 没什么问题，正常的查询在1-20ms之内**

  ```
  网上测试参考
  	8G内存 2核cpu      
  	最大的吞吐量为每秒写入60万条数据（3个字段，1个索引）   读取性能 2ms/条
  	
  环卫云项目参考  
  	32G   16核cpu  1000Mbps的带宽
  	平均吃  8G内存   600% cpu 
  	每秒写入  2-3K条数据（50个字段、2-3个索引）
  ```

  

- 功能性
  - HTTP协议的API 
  - 通过数据库本身的聚合函数
  - retention policy：数据存储策略（默认策略为autogen）InfluxDB没有删除数据操作，规定数据的保留时间达到清除数据的目的
  - 底层时间戳为纳秒， 存储用的是utc时间戳（自1970年1月1日起的相对时间戳），支持通过  tz（Asia/China）转化成对应时区的时间

- 横向对比
  - 在时序数据库出现之前，存储时序数据这项领域一直被 MongoDB 所占据，在数据量不够高的情况下，InfluxDB 并不会比 MongoDB 或者 ElasticSearch 有更明显的优势
  - MySQL、Oracle 等，优点是具有 ACID 的特性，各方面能力都比较均衡。缺点是查询、插入和修改的性能都很一般。
  - 基于 PostgreSQL 的 TimeScaleDB
  - 高性能写入、高性能查询、低存储成本、支持海量时间线、弹性

- 纵向对比
  - 数据库名、存储策略、 measurement （类似 mysql 的表）名与 tag 名一起作为时间轴的标记 (series)，series是存在内存中的，所以不要太多series，影响性能


## 2 [部署安装](https://gitee.com/ming-liang-zhou/influxdb/blob/master/influxdb.md)

```
默认配置
默认influxDB使用以下端口
8086: 用于客户端和服务端交互的HTTP API
8088: 用于提供备份和恢复的RPC服务

配置文件path 
 /etc/influxdb/influxdb.conf

密码
 默认用户名、密码是admin、password
 
 
 #influx 进入influx cmd
 influx
 #添加一个管理员用户
 create user "root" with password 'root' with all privileges

默认时间戳
   时间戳默认用的utc, db底层存储的单位是纳秒（等值查询时） 1685943000000000 
   influxDB使用所在主机的本地时间的UTC时间(比国内晚8个小时)来设置timestamp
```



QA:

​    1.chronograf的端口8888被占用，修改配置文件后重启

```

经过测试后，需要修改以下文件中的端口配置才能修改默认端口
/usr/lib/systemd/system/chronograf.service
/usr/lib/chronograf/scripts 中的chronograf.service 和init.sh文件中的端口

执行更新命令
systemctl daemon-reload
重新启动chronograf后可以正常打开
sudo systemctl start chronograf

```



## 3 Cmd



```
# service方式
# 启动命令 service influxdb start
# 重启命令 service influxdb restart
# 关闭命令 service influxdb stop
# 查看启动日志 sudo journalctl -u influxdb.service -f

# systemctl 方式
# 启动命令 sudo systemctl start influxdb
# 关闭命令 sudo systemctl stop  influxdb



#查看influxdb进程
ps -ef|grep influxdb

#杀进程
kill -9 进程id

#其他参考
#启动influxdb
nohup /mnt/app/mydata/influxdb/influxdb-1.8.10-1/usr/bin/influxd -config /mnt/app/mydata/influxdb/influxdb-1.8.10-1/etc/influxdb/influxdb.conf >>/mnt/app/mydata/influxdb/influxdb-1.8.10-1/var/log/influxdb/nohup.log 2>&1 &


#可视化面板相关
#查看chronograf进程
ps -ef|grep chronograf
#杀进程
kill -9 进程id
#启动chronograf
nohup /mnt/app/mydata/influxdb/chronograf-1.9.4-1/usr/bin/chronograf >> /mnt/app/mydata/influxdb/chronograf-1.9.4-1/var/log/chronograf/nohup.log 2>&1 &

nohup /tmp/zjf/chronograf-1.9.4-1/usr/bin/chronograf >> /tmp/zjf/chronograf-1.9.4-1/var/log/chronograf/nohup.log 2>&1 &
```



\##https://www.metabase.com/docs/latest/installation-and-operation/running-metabase-on-docker

http://172.16.112.9:3000/  cowa2022..



```
./influx -ssl -unsafeSsl -username iot -password cowa_iot_influx


## https://docs.influxdata.com/influxdb/v1.8/
## https://jasper-zhang1.gitbooks.io/influxdb/content/
1.创建数据库iot
create database iot;
2.使用数据库
use iot;
3.查询所有measurement
show measurements;
4.创建管理员用户admin
create user "admin" with password '密码' with all privileges;
5.创建普通用户iot
create user "iot" with password '密码';
6.创建普通用户iotRead
create user "iotRead" with password '密码';
7.给iot用户授权iot数据库只写权限
grant write on "iot" to "iot";
8.给iotRead用户授权iot数据库只读权限
grant read on "iot" to "iotRead";
9.创建默认存储策略（rp_30d），数据保留近30天
create retention policy "rp_30d" on "iot" duration 30d replication 1 default;
10.创建存储策略（rp_10d），数据保留近10天
create retention policy "rp_10d" on "iot" duration 10d replication 1;
11.创建存储策略（rp_3d），数据保留近3天
create retention policy "rp_3d" on "telegraf" duration 3d replication 1;
12.连续查询
#mydb 中新建了一个名为 cq_30m 的连续查询，每三十分钟取一个used字段的平均值，加入 mem_used_30m 表中
create continuous query cq_30m on mydb begin select mean(used) into men_used_30s from men group by time(30m) end;
# 查看所有的连续查询
show continuous queries;
# 删除mydb 数据库中的连续查询 cq_30m
drop continuous query cq_30m  on  mydb
13.常用函数
#count()函数 返回一个（field）字段中的非空值的数量
select count(*) from disk_free

#DISTINCT()函数 返回一个字段（field）的唯一值，函数里面是 field 值！
select distinct(value) from disk_free

#MEAN() 函数 返回一个字段（field）中的值的算术平均值（平均值）字段类型必须是长整型或float64
select mean(value) from disk_free

#MEDIAN()函数 从单个字段（field）中的排序值返回中间值（中位数）
#字段值的类型必须是长整型或float64格式
select median(value) from disk_free

#SPREAD()函数 返回字段的最小值和最大值之间的差值
select spread(value) from disk_free

#SUM()函数 返回一个字段中的所有值的和
select sum(value) from disk_free

#TOP()函数 返回一个字段中最大的N个值
select top("value", 1) from disk_free

#BOTTOM()函数 返回一个字段中最小的N个值
select bottom("value",2) from disk_free

#FIRST()函数 返回一个字段中存活时间最长的值
select first(value) from disk_free

#MAX()函数 返回一个字段中的最大值
#MIN()函数 返回一个字段中的最小值
select max(value) from disk_free
select min(value) from disk_free

#PERCENTILE()函数 返回排序值排位为N的百分值
#value 字段按照不同的 value求百分比，然后取第五位数据。
select percentile(value,99),value from disk_free

PERCENTILE(<field_key>,100)相当于MAX(<field_key>)。
PERCENTILE(<field_key>, 50)几乎等同于MEDIAN(<field_key>)，但MEDIAN()如果字段键包含偶数个字段值，则该函数返回两个中间值的平均值。
PERCENTILE(<field_key>,0)不等于MIN(<field_key>)。

FAQ：
1.一个点由measurement名称，tag set和timestamp唯一标识。如果您提交具有相同measurement，tag set和timestamp，但具有不同field set的行协议，则field set将变为旧field set与新field set的合并，并且如果有任何冲突以新field set为准。

```

## 4 sql操作

```
插入
    tag 字段插入后不能删除，慎重！
    field字段插入后不能删除，慎重！
    
    
delete
    influxdb作为时序数据库的一种，是不提供类似mysql的delete操作，也就是，原则上，你删不了数据，除非删库。 但是，influxdb提供了数据保留策略policies，通过对policies操作，可以达到删除数据的效果(1)。可以根据time和tag进行批量删除，不支持基于field进行批量删除，只能一条一条删除(2)。DELETE SQL删除数据的过程是：根据DELETE SQL中指定的WHERE条件，从tsi/tsl文件中过滤得到满足条件的seriesID，通过查找series index以及series segment文件得到seriesID对应的seriesKey(3)。
```



```
基础数据

查询数据库大小
select sum(diskBytes) / 1024 / 1024 /1024 from _internal."monitor"."shard" where time > now() - 10s
select sum(diskBytes) / 1024 / 1024 /1024 from _internal."monitor"."shard" where time > now() - 10s group by "database"

查询记录条数
SELECT count(*) FROM "dataAnalysis"."autogen"."carGis" 
SELECT count(*) FROM "dataAnalysis"."autogen"."carGis" where time>'2022-09-22 12:00:00'  and  time<'2022-09-23 00:00:00'  tz('Asia/Shanghai')     中国时区---东八
SELECT count(*) FROM "dataAnalysis"."autogen"."carGis" where time>'2022-09-22 12:00:00'  and  time<'2022-09-23 00:00:00'                                    UTC时区---东零


查看数据库的表字段类型
  show  field keys from dataAnalysis

设置当前策略为default策略
ALTER RETENTION POLICY "rp_30d" ON "iot" DEFAULT


```



### query

- time特殊                 不需要''或者""包起来   直接time
- tag和fields              需要""包起来   

​         "tagkey"='30.22'   等号右边，若为字符串需要写成单引号

​         如果写错了，则查询会没有结果

- 默认不写 tz('Asia/Shanghai')  表示查询的时间戳为 UTC，通常需要在最后加 tz('Asia/Shanghai')

```

查询部分字段
SELECT "latitude", "longitude" FROM "iot"."rp_30d"."drive" WHERE time > :dashboardTime: AND time < :upperDashboardTime: AND "carNo"='80001' AND "longitude"='NaN'

查询全部字段
SELECT * FROM "dataAnalysis"."autogen"."carGis" where time>'2022-09-22 00:00:00'  and  time<'2022-09-23 00:00:00'  tz('Asia/Shanghai') 

时区
utc查询和 tz时区查询
 time>'2022-09-22 00:00:00'  and  time<'2022-09-23 00:00:00'  
等价于
 time>'2022-09-22 08:00:00'  and  time<'2022-09-23 08:00:00'  tz('Asia/Shanghai')

```

分页查询



### **Insert**

- measurement加tags组成了一个series 
- tag的key和value都必须是字符串。
-  fields的key也是必须的，而且是字符串，默认情况下field的value是float类型的。
-  tag+timestamp的组合--->唯一定义1条记录，即相同time相同tag的唯一记录 

重复插入后面的会默认覆盖之前的记录

2  不支持update 重新插入相同记录来达到修改的效果

```Java
USE "zml_test"."rp_30d"; insert "zml_test"."rp_30d"."aaaaa"  "carno"='123' "bbb"='111' 

USE "zml_test"."rp_30d"; insert "zml_test"."rp_30d"."zmlzzz"  "dev_id"='1' "aa"='22' "bb"='111' time=1690767675736
```



### **delete**

- 不支持删除tag或者field
- 通过retency来实现数据删除
- ` `表存储策略上：默认autogen无法删除数据  只能insert覆盖
- 非autogen策略表可以删除
- 支持删除time段的数据



```

1 根据时间点  精确到ns
    delete from "dataAnalysis"."rp_7d"."carGisT" where time=1688612365000
2 根据时间范围
    delete from "dataAnalysis"."rp_7d"."carGisT" where time='2022-10-19 17:31:43.762'  tz('Asia/Shanghai') 
    DELETE FROM <measurement_name> WHERE [<tag_key>='<tag_value>'] | [<time interval>]

3 删除某表的指定时间范围的数据
    delete from  "dataAnalysis"."rp_7d"."carGisT" where time<'2022-10-19 17:31:43.762Z' 
    And time<'2022-10-19 17:31:43.762Z  tz('Asia/Shanghai') 

4 删除某个tag的所有记录
    USE "zml_test"."rp_30d"; DROP SERIES FROM "mbb" WHERE "ccc" = '111'

5 删除某个tag的时间范围内的记录(UTC)
    USE  "iot"; DELETE FROM "drive" WHERE "carNo" = '1122333222' AND time > '2022-10-19 07:40:23'   AND time < '2022-10-19 19:40:23'   
    USE  "dataAnalysis"; DELETE FROM "carGisT" WHERE "carNo" = '1122333222' AND time= '2022-10-19 07:40:23'
    hds

6 删除整表
    USE "recloud_test";  drop  measurement xxx  删除整表
    USE "recloud_test";  delete from xxx   删除的是recloud_test下所有策略的measurmentx   
    USE "iot";  delete from car_detail

```



时差问题

chronograf查询----勾选local-----时间要减少8h

对应mysql

SELECT count(*) FROM "dataAnalysis"."autogen"."carGis" 

SELECT count(*) FROM "dataAnalysis"."autogen"."carGis" where time>'2022-09-22 12:00:00'  and  time<'2022-09-23 00:00:00'  tz('Asia/Shanghai')

联表查询

```
SELECT "height", "latitude" FROM "iot"."rp_30d"."drive" WHERE time > :dashboardTime: AND time < :upperDashboardTime: AND "carNo"='14001'


SELECT * FROM "iot"."rp_30d"."drive"，"iot"."rp_30d"."gns" WHERE time > :dashboardTime: AND time < :upperDashboardTime: AND "carNo"='14001'
```

查询某段时间范围内的距今最近的1条

```
select * from recloud_test.rp_30d.iot_carinfo_flat where time > '2023-06-26 20:00:20' AND time < '2023-06-26 20:00:21' And \"carNo\"='14003' ORDER BY time Desc LIMIT 1 tz('Asia/Shanghai') 


select * from recloud_test.rp_30d.car_detail where time > '2023-06-25 20:00:20' AND time < '2023-06-26 23:00:21'  ORDER BY time Desc LIMIT 1 tz('Asia/Shanghai') 
```

查询 某字段不为空

```
select * from recloud_test.rp_30d.iot_carinfo_flat where And \"carNo\"='14003' ORDER BY time Desc LIMIT 1 tz('Asia/Shanghai') 
```

SELECT x.* FROM cariot.iot_car_gis_csq_show x

WHERE car_time between "2022-10-01 00:00:00" and "2022-10-01 23:59:59" and car_no ='80017

insert数据覆盖

```
List<IotCarGisCsqShowDo> doList = Lists.newArrayList();
IotCarGisCsqShowDo iotCarGisCsqShowDo = new IotCarGisCsqShowDo();
iotCarGisCsqShowDo.setBdLongitude(118.397373534543);
iotCarGisCsqShowDo.setBdLatitude(31.39545887027735);
iotCarGisCsqShowDo.setLatitude(31.3915865);
iotCarGisCsqShowDo.setLongitude(118.3853872);
iotCarGisCsqShowDo.setCsq(30.0);
LocalDateTime of = LocalDateTime.of(2022, 10, 12, 5, 28, 50);
System.out.println(of.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli());
iotCarGisCsqShowDo.setCarTime(of);
doList.add(iotCarGisCsqShowDo);
BatchPoints batchPoints = BatchPoints.database("dataAnalysis")
        .retentionPolicy("autogen")
        .precision(TimeUnit.MILLISECONDS)
        .consistency(InfluxDB.ConsistencyLevel.ANY)
        .tag("carNo", "80017")
        .points(doList.stream().map(i -> Point.measurement("carGis").addFieldsFromPOJO(i).time(i.getCarTime().atZone(ZoneId.systemDefault()).toInstant().toEpochMilli(), TimeUnit.MILLISECONDS).build())
                .collect(Collectors.toList())
        ).build();
influxDBService.batchInsert(batchPoints);
```
