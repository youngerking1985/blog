# rocksdb hello 分析之*：MemTable源码解读
本章节主要研究rocksdb put流程，相关概念，源码等。
- MemTable主要功能和使用场景
- MemTable Put流程
- MemTable Get流程

---
## 测试代码

## 主要相关类解释
MemTable：内存表的封装类，封装了具体的实现
MemTableRep：内存表的实现类
MemTableIterator：
MemTableBackwardIterator
MemTableList
Cleanable：抽象类，

## 流程说明
### Add关键流程
WriteBatchInternal::InsertInto->
MemTable::Add

### Get关键流程

## 相关类，函数实现
Ref、Unref
GET
Add
Update
MarkImmutable

## 结论

### 待整理

## 疑问
1. MemTable结构跟SSTFile差异？
2. 一个ColumnFamily是否对应一个MemTable，是否是一对一关系
3. 写入时，value size是uint32，也就是大小不能超过4GB？

## stack
1. from put
2. MemTable
3. 二叉树原理及实现，B Tree，SkipList等性能对比；
4. LRUCache, best实现、pg实现、rocksdb实现
5. 图、DFS、BFS、Dijkstra算法；
6. MemTable
8. Iterator，MemTableIterator，
9. 实现一个移动构造函数，并测试

## pop

---
## 参考
1. 官方文档（推荐）：https://github.com/facebook/rocksdb/wiki/MemTable
2. 相关类结构和职责（推荐）：https://www.jianshu.com/p/9e385682ed4e
3. MySQL · RocksDB · Memtable flush分析 https://developer.aliyun.com/article/643754
4. 重点将跳表内容：https://whoiami.github.io/ROCKSDB_MEMTABLE

