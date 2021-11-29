# citus mx使用
## 概述
在部分场景下，如时序数据入库，有大量的数据需要插入，这种情况下，citus的CN会成为瓶颈，约为计算Plan会占用大量的CPU，所以增加worker，无法有效提升数据插入效率。这种情况下可以使用MX模式。
> MX模式原理是将CN的元数据信息，复制到Worker上，将执行计划的生产分担到对应的Worker节点中
> MX模式下，worker可以支持DML操作，但不支持DDL操作。
## 使用流程
### 配置citus分布式环境
参考 citus基本试使用
### 配置节点流复制
需要在CN节点上，设置流复制模式，通过修改postgresql.conf文件
``` shell
citus.replication_model = streaming
```
### 添加节点(当前CN 4.18)
``` sql
SELECT citus_add_node('192.168.4.153', 5432);
-- 添加其他节点
```

### 创建测试表
``` sql
create table tb(id int primary key,c1 int,c2 text);
SELECT create_distributed_table('tb','id');
```

### 同步元数据
``` sql
SELECT start_metadata_sync_to_node('192.168.4.153', 5432);
```

### 数据插入
可在4.18和4.153同时执行insert语句

## 测试数据
使用iBEST-DB轨迹模型进行验证
### 测试环境
4.18：CN节点
4.18(5431)：worker节点
4.153：worker节点
4.154：worker节点

> 每个节点包含48Core，96GB等；存储采用4块Sata盘组成的RAID5

### 单机环境
插入实时轨迹，在不插入实时轨迹的情况下，每秒约为11w/s

### 分布式环境
插入实时轨迹，在不插入实时轨迹的情况下，单机每秒约为7w；3节点每秒约为21w

## 疑问
### 1. 执行元数据同步到worker后，分片表隐藏到哪里了？
可通过视图查看，citus_shards_on_worker，citus_shard_indexes_on_worker
或者citus.override_table_visibility = false

### 2. worker分片不存储数据设置
``` sql
SELECT  master_set_node_property('127.0.0.1', 9001, 'shouldhaveshards', true);
```

## 参考
1. 再谈Citus 多CN部署与Citus MX：https://github.com/ChenHuajun/blog_xqhx/blob/main/2020/2020-07-05-%E5%86%8D%E8%B0%88Citus%20%E5%A4%9ACN%E9%83%A8%E7%BD%B2%E4%B8%8ECitus%20MX.md
2. 多CN citus发行命令记录：https://github.com/ChenHuajun/blog_xqhx/blob/main/2018/2018-10-04-%E5%A4%9ACN%20citus%E5%8F%91%E8%A1%8C%E5%91%BD%E4%BB%A4%E8%AE%B0%E5%BD%95.md#71-%E5%B8%A6%E4%BA%8B%E5%8A%A1%E7%9A%84sql%E5%A4%9A%E5%88%86%E7%89%87%E5%AD%97%E6%AE%B5%E7%9A%84%E6%9B%B4%E6%96%B0
3. citus实战系列之四多CN部署：https://github.com/ChenHuajun/blog_xqhx/blob/main/2018/2018-06-11-citus%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E4%B9%8B%E5%9B%9B%E5%A4%9ACN%E9%83%A8%E7%BD%B2.md


