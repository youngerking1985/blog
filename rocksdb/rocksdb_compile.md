# rocks相关编译安装
## 一、rocksdb编译安装
参考官方编译指南即可
1. https://github.com/facebook/rocksdb/blob/master/INSTALL.md
编译gfflags时，要添加fPIC参数
cmake -DCMAKE_CXX_FLAGS=-fPIC ../

2. 压缩包可根据需求进行编译，不过如果要使用db_bench，需要编译snappy，
``` cmake ../ -DWITH_SNAPPY=1 ``` 

3. rocksdb默认编译为debug模式，静态库约660M，动态库约200M，release模式，静态库21M，动态库9M，release编译设置：
``` cmake ../  -DCMAKE_BUILD_TYPE=Release ```

问题：
1. db_bench需要snappy，但是编译时提示，Could NOT find Snappy (missing: Snappy_DIR)，这个提示不用管，只要确定snappy-devel已经安装即可
2. 


## 二、pgrocks编译安装
网上默认版本不支持pg13，下载youngerking1985下的[fork版本](https://github.com/youngerking1985/pgrocks-fdw)，支持cmake和makefile在pg13下的编译；
> 注意要修改配置文件重启

## 三、测试
可运行目录下的examples

## 参考
1. rocksdb官方编译说明： https://github.com/facebook/rocksdb/blob/main/INSTALL.md
2. postgres外部表： https://github.com/youngerking1985/pgrocks-fdw
