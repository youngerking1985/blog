# rocksdb hello 分析之二：put流程源码解读
本章节主要研究rocksdb put流程，相关概念，源码等。
- 打开、put、关闭，文件变化情况
- 再次打开、read、关闭，文件变化情况
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
-rw-r--r--. 1 root root    16 Sep 15 06:40 CURRENT
-rw-r--r--. 1 root root    37 Sep 15 06:40 IDENTITY
-rw-r--r--. 1 root root     0 Sep 15 06:40 LOCK
-rw-r--r--. 1 root root 22098 Sep 15 06:41 LOG
-rw-r--r--. 1 root root    57 Sep 15 06:41 MANIFEST-000004
-rw-r--r--. 1 root root  6209 Sep 15 06:40 OPTIONS-000007
```

## 生成的文件说明
### 术语解释

### 文件内容解读


## 相关类，函数实现
DB
DBImpl
StackableDB
CompactedDBImpl
LogFile(WalFile)

### 文件：


## 数据库文件打开、删除

## 写入

## 查询

## 不同的数据库打开模式
### 默认DB::Open
### 只读模式
多线程读

## 结论
1. 

## 疑问
1. 社区有个问题，说是写入之后需要重启，其他reader才能读取到，https://github.com/facebook/rocksdb/issues/908，不过最新回答是已经做了这块的工作，待确认？
- 经测试，写入不需要重启，其他reader也可以读到

## TODO
1. options解读
2. 

---
## 参考
1. RocksDB——Put: https://www.jianshu.com/p/daa18eebf6e1