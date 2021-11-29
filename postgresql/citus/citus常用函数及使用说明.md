# citus常用函数及使用说明

### func
描述
#### 语法

#### 参数

#### 示例

## 分布式配置

### citus_add_node
描述
#### 语法

#### 参数

#### 示例

### citus_remove_node
从pg_dist_node 元数据表中移除worker节点。如果该worker上还有数据分片执行将会报错，所以执行该函数前，应该先删掉该节点上的数据分片
#### 语法
```
void citus_remove_node(nodename text, nodeport integer)
```
#### 参数
**nodename:** 节点地址，ip或者dns

**nodeport:** 加入时的端口号

#### 示例

## 创建修改（DDL）

### create_distributed_table
用于定义分布式表，如果是哈希分布式表，则创建其分片。 此函数接受表名、分布列和可选的分布方法，并插入适当的元数据以将表标记为已分布。 如果未指定分配方法，则该函数默认为“哈希”分配。 如果表是哈希分布的，该函数还会根据分片计数和分片复制因子配置值创建工作分片。 如果表包含任何行，它们将自动分发到工作节点。

#### 语法
```
 void create_distributed_table(table_name regclass,
                                                    distribution_column text,
                                                    distribution_type citus.distribution_type DEFAULT 'hash',
                                                    colocate_with text DEFAULT 'default',
                                                    shard_count int DEFAULT NULL);
```
#### 参数
**table_name:** 需要进行分布式的原始表.

**distribution_column:** 用于进行分布式的表字段，会根据表字段哈希，将数据分到不同的分片上

**distribution_type:** 可选参数，定义表分布式的方法，包括append和hash模式，默认是hash模式

**colocate_with:** 可选参数，将当前表包含在另一个表的并置组中。 默认情况下，当表由相同类型的列分布、具有相同的分片计数和相同的复制因子时，它们会位于同一位置。 如果以后想打破这种托管，可以使用update_distributed_table_colocation。 colocate_with 的可能值是默认值，none 以启动新的并置组，或与该表共置的另一个表的名称。 （见共同定位表。）

请记住， colocate_with 的默认值会隐式并置。 正如 Table Co-Location 解释的那样，当表相关或将被连接时，这可能是一件好事。 但是，当两个表不相关但碰巧对它们的分布列使用相同的数据类型时，意外地将它们放在一起会降低分片重新平衡期间的性能。 表碎片将在“级联”中不必要地移动在一起。 如果你想打破这种隐式托管，你可以使用 update_distributed_table_colocation。

如果新的分布式表与其他表不相关，最好指定 colocate_with => 'none'。

**shard_count:**  可选，创建新的分布式表的分片数量，分片数量取值范围为1-64000，

#### 示例
```
create table tb1(id int primary key, c1 int);
SELECT create_distributed_table('tb1', 'id');

-- alternatively, to be more explicit:
SELECT create_distributed_table('tb1', 'repo_id', 'hash', 'default', 4);
```

### truncate_local_data_after_distributing_table

### alter_distributed_table

## 疑问
### 1. master_add_node和citus_add_node区别？
master_add_node里实际执行了citus_add_node，citus官方示例中使用的是citus_add_node，所以后面使用应该尽可能用citus_***

## 参考
1. 官方文档： https://docs.citusdata.com/en/v10.2/develop/api_udf.html

## 待整理
### 创建shard_count计算函数