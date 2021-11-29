
# RocksDB 源码分析之三：Cleanable类解读
本章节主要研究Cleanable类，主要知识点包括：
1. 设计目的
2. 运行方式
3. 使用测试

---
## 测试代码

## 概述
Cleanable类是一个用于链表内存回收的基类，被Iterator、InternalIteratorBase、PinnedIteratorsManager、PinnableSlice等类继承
采用删除拷贝构造函数方式，避免用户直接创建Cleanable对象，导致多次复制导致的效率问题

## 术语解释
Slice
Cleanup：结构体
CleanupFunction：负责进行数据资源释放的函数指针


## 流程说明

## 相关文件
1. Cleanable声明在cleanable.h文件中
2. 实现在iterator.cc文件中
3. 测试用例在cleanable_test.cc

## 相关类，函数实现
DelegateCleanupsTo：将当前对象连接到另一个Cleanble对象链表中
RegisterCleanup：配合DelegateCleanupsTo使用，构建链表
DoCleanup：执行链表节点的内存释放，通过调用函数指针


## 结论


### 待整理

## 疑问

## stack
1. from memtable
2. 单机和分布式提交测试；2pc提交；（效率、安全）
3. 

## pop

---
## 参考
1. 
