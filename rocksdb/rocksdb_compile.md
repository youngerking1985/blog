# rocks相关编译安装
## 一、rocksdb编译安装
参考官方编译指南即可
1. https://github.com/facebook/rocksdb/blob/master/INSTALL.md
编译gffalgs时，要添加fPIC参数
cmake -DCMAKE_CXX_FLAGS=-fPIC ../


## 二、pgrocks编译安装
网上默认版本不支持pg13，下载youngerking1985下的fork版本，支持cmake和makefile在pg13下的编译；
注意要修改配置文件重启