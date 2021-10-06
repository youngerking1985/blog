# rocksdb hello 分析之二：put写入流程源码解读
本章节主要研究rocksdb put流程，相关概念，源码等。
- 打开、put、关闭，文件变化情况；及关键函数或调用
- 再次打开、read、关闭，文件变化情况；及关键函数及调用
- GET过程分析；
- put什么时候会出发merge（此问题可能放在下一个章节）
---
## 测试代码
在空目录下，执行DB::Open操作后的put及关闭操作
``` c++
    //创建rocks目录
    DB* db;
    Options options;
    options.create_if_missing = true;

    std::string dirPath = "/home/rocksdata";
    Status s = DB::Open(options, dirPath, &db);
    assert(s.ok());

    //持续写入
    for(int i = 0; i < 10000; i ++)
    {
        std::chrono::milliseconds dur(200);
        std::this_thread::sleep_for(dur);
        db->Put(WriteOptions(), "key" + std::to_string(i), "value" + std::to_string(i));
        assert(s.ok());
        std::cout << "put key" << i << std::endl;
    }

    delete db;
```
空目录下执行后产生的文件
``` shell
-rw-r--r--. 1 root root     0 Sep 15 06:40 000005.log

```

## 流程说明
### 写入流程
流程图（待画图）
1. 将一条或者多条操作的记录封装到WriteBatch
2. 将记录对应的日志写到WAL文件中
3. 将WriteBatch中的一条或者多条记录写到内存中的memtable中


## 生成的文件说明
### 术语解释


## 术语解释
Slice
PinnableSlice

## 相关类，函数实现
DB
DBImpl
StackableDB
CompactedDBImpl
LogFile(WalFile)
Slice
WriteBatch
VersionSet
ColumnFamilySet
MemTableRep
MemTable

##### put调用流程待整理：
DB::Put->DBImpl::Put->DB::Put->DBImpl::Write->DBImpl::WriteImpl
->DBImpl::WriteToWAL->log::Writer::AddRecord->WritableFileWriter::Append->WritableFileWriter::WriteBuffered->FSWritableFile::Append->->WritableFileWriter::Flush
db_impl_write.cc DB::Put，不传Family参数，会调用默认Family构造参数
    return Write(opt, &batch);
    WriteImpl
IOStatus DBImpl::WriteToWAL
  //写入wal文件
     io_s = WriteToWAL(*merged_batch, log_writer, log_used, &log_size);
     IOStatus io_s = log_writer->AddRecord(log_entry);

##### memtable调用
初始化memtable
DBImpl::Open->DBImpl::Recover->VersionSet::Recover->
VersionEditHandler::CreateCfAndInit
VersionSet::CreateColumnFamily
ColumnFamilyData::ConstructNewMemtable

写入memtable
DBImpl::WriteImpl->
WriteBatchInternal::InsertInto->
MemTable::Add


### 文件：


## 数据库文件打开、删除

## 写入

## 查询

## 不同的数据库打开模式
### 默认DB::Open
### 只读模式
多线程读

## 结论


### 待整理
1. MemTable也是一种ColumnFamily
2. DB打开的时候，会创建一个MemTable
3. 数据put过程中，会先写入到wal中，然后再将数据写入到memtable中；

## 疑问
1. 社区有个问题，说是写入之后需要重启，其他reader才能读取到，https://github.com/facebook/rocksdb/issues/908，不过最新回答是已经做了这块的工作，待确认？
- 经测试，写入不需要重启，其他reader也可以读到
2. max_write_buffer_size默认大小为64MB，是否到了64MB，会出发合并
3. key是怎么做排序的？
4. 如果插入的key在sst文件已经存在，是如何进行merge的？
5. kBlockSize默认值是多少？
6. 写入的时候，key和value的slice，最后只看到一个？
7. 怎么获取key的数量和具体key？

## stack
1. 将Slice Batch数据打包流程看明白
2. WriteBatch作用？
3. MemTable
   - ImmutableMemTable,
4. MemTable

---
## 参考
1. RocksDB——Put: https://www.jianshu.com/p/daa18eebf6e1
2. write流程
   - https://zhuanlan.zhihu.com/p/315333301
   - https://zhuanlan.zhihu.com/p/343323703
   - https://zhuanlan.zhihu.com/p/156831542
3. 