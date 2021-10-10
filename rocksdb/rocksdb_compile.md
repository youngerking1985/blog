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

## 四、基于飞腾麒麟环境的编译记录（Arm架构）
环境说明：Kylin 4.0.2；FT2000PLUS
1. 该版本系统应该是采用ubuntu，需要通过apt-get进行包安装
2. 编译采用的rocksdb版本为6.22；系统已经自带gcc，g++等
### 4.1 更新cmake
3. 系统默认环境cmake版本较低，需要重新编译cmake，配置cmake过程中发现需依赖libssl-dev，默认配置源没有该源
4. 配置libssl-dev源，可参考https://blog.csdn.net/hknaruto/article/details/104654406，配置好后，通过```sudo apt-get update```更新源
5. 安装ssl等，```sudo apt-get install openssl libssl-dev```安装ssl
6. 配置，编译
```
sudo ./configure
sudo make -j 64
sudo make install
sudo rm /usr/bin/cmake
sudo ln -s /usr/local/bin/cmake /usr/bin/cmake
cmake --version
```
### 4.2 编译rocksdb
7. 安装snappy（不安装也可以），```sudo apt-get install libsnappy-dev```
8. 编译，如果没有安装snappy，删除-DWITH_SNAPPY=1
``` shell
sudo cmake ../  -DCMAKE_BUILD_TYPE=Release -DWITH_SNAPPY=1
sudo make -j 64
sudo make install
```

## 参考
1. rocksdb官方编译说明： https://github.com/facebook/rocksdb/blob/main/INSTALL.md
2. postgres外部表： https://github.com/youngerking1985/pgrocks-fdw
