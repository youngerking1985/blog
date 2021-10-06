# rocksdb 性能测试：db_bench（待完善）
本章节主要研究rocksdb benchmark测试

---
## 测试代码

## 概述
rocksdb提供benchmark工具来对自身性能进行全方位各个纬度的评估，包括顺序读写，随机读写，热点读写，删除，合并，查找，校验等性能的评估，十分好用。

## 常用参数及示例
参数 | 作用 | 默认值 | 示例
---|---|---|---
db | 数据库文件路径 | 内存路径如//tmp/.. | --db=/home/temp
disable_wal | 不写wal | false |--disable_wal=false
value_size | value大小，byte| 100|--value_size=1000
num|kv数量|1000000|--num=10
readonly |只读模式|false|--readonly=true
threads|线程数量|1|--threads=16

### Benchmarks List:
benchmarks可以多个配置组着使用，多个配置将按照顺序执行，示例:
```
./db_bench --benchmarks="fillseq,readrandom,readseq"
```
      	fillseq       -- write N values in sequential key order in async mode
      	fillseqdeterministic       -- write N values in the specified key order and keep the shape of the LSM tree
      	fillrandom    -- write N values in random key order in async mode
      	filluniquerandomdeterministic       -- write N values in a random key order and keep the shape of the LSM tree
      	overwrite     -- overwrite N values in random key order in async mode
      	fillsync      -- write N/100 values in random key order in sync mode
      	fill100K      -- write N/1000 100K values in random order in async mode
      	deleteseq     -- delete N keys in sequential order
      	deleterandom  -- delete N keys in random order
      	readseq       -- read N times sequentially
      	readtocache   -- 1 thread reading database sequentially
      	readreverse   -- read N times in reverse order
      	readrandom    -- read N times in random order
      	readmissing   -- read N missing keys in random order
      	readwhilewriting      -- 1 writer, N threads doing random reads
      	readwhilemerging      -- 1 merger, N threads doing random reads
      	readrandomwriterandom -- N threads doing random-read, random-write
      	prefixscanrandom      -- prefix scan N times in random order
      	updaterandom  -- N threads doing read-modify-write for random keys
      	appendrandom  -- N threads doing read-modify-write with growing values
      	mergerandom   -- same as updaterandom/appendrandom using merge operator. Must be used with merge_operator
      	readrandommergerandom -- perform N random read-or-merge operations. Must be used with merge_operator
      	newiterator   -- repeated iterator creation
      	seekrandom    -- N random seeks, call Next seek_nexts times per seek
      	seekrandomwhilewriting -- seekrandom and 1 thread doing overwrite
      	seekrandomwhilemerging -- seekrandom and 1 thread doing merge
      	crc32c        -- repeated crc32c of 4K of data
      	xxhash        -- repeated xxHash of 4K of data
      	acquireload   -- load N*1000 times
      	fillseekseq   -- write N values in sequential key, then read them by seeking to each key
      	randomtransaction     -- execute N random transactions and verify correctness
      	randomreplacekeys     -- randomly replaces N keys by deleting the old version and putting the new version
        timeseries            -- 1 writer generates time series data and multiple readers doing random reads on id



## 常用测试场景
1. 只读测试
注意，要设置 -use_existing_db=true ，否则就算存在该数据库，默认情况下将会创建一个空数据库替代原数据库
```
./db_bench --db=/home/temp/rocks --benchmarks=readrandom --readonly --use_existing_db=true
```

## 详细参数
Slice

## 对比测试
测试环境，5.194；虚拟机16Core，32GB内存，虚拟存储500GB; time write 766 MB/s, read 800MB/s，bs 1024k
### 写wal和不写wal效率差异
根据测试，不写wal约为写wal的三倍左右
```
./db_bench: 
fillseq      :       3.322 micros/op 300985 ops/sec;   33.3 MB/s
./db_bench -disable_wal
fillseq      :       1.195 micros/op 836521 ops/sec;   92.5 MB/s
```
### 随机写和顺序写效率差异
顺序写能比随机写高50%左右
```
./db_bench --db=/home/temp/rocks --value_size=100 --benchmarks="fillseq"
fillseq      :       3.668 micros/op 272606 ops/sec;   30.2 MB/s
./db_bench --db=/home/temp/rocks --value_size=100 --benchmarks="fillrandom"
fillrandom   :       5.497 micros/op 181919 ops/sec;   20.1 MB/s
```
### 内存写和磁盘写效率差异
在io没有成为瓶颈之前，磁盘和内存效率相当
```
./db_bench --db=/home/temp/rocks
fillseq      :       3.144 micros/op 318083 ops/sec;   35.2 MB/s
```
### 顺序写的情况下，随机读和顺序读效率差异
./db_bench --db=/home/temp/rocks --benchmarks="fillseq,readrandom,readseq"
> 清除系统缓存，效率差异不大

```
fillseq      :       4.016 micros/op 249017 ops/sec;   27.5 MB/s
readrandom   :       7.672 micros/op 130339 ops/sec;   14.4 MB/s
readseq      :       0.438 micros/op 2283516 ops/sec;  252.6 MB/s
```
### 随机写情况下，随机读和顺序读效率差异
./db_bench --db=/home/temp/rocks --benchmarks="fillrandom,readrandom,readseq"
> 都是顺序读，在顺序写的情况下，效率比随机写的快50%，不管哪种情况，顺序读都比随机快10倍左右

```
fillrandom   :       5.570 micros/op 179522 ops/sec;   19.9 MB/s
readrandom   :      10.559 micros/op 94704 ops/sec;   10.5 MB/s
readseq      :       0.644 micros/op 1551782 ops/sec;  171.7 MB/s
```


## 流程说明


## 相关类，函数实现



## 结论


### 待整理

## 疑问
1. 设置value大小，key大小
2. 设置key数量
3. 设置文件路径
4. 设置线程数量（写入、读取）？
5. 只读测试？
6. 写入时，多线程是如何避免冲突？

## stack



## pop

---
## 参考
1. 官方说明：https://github.com/facebook/rocksdb/wiki/Benchmarking-tools
2. https://github.com/facebook/rocksdb/wiki/RocksDB-In-Memory-Workload-Performance-Benchmarks