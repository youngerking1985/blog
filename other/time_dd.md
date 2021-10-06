# 基于dd命令，进行硬盘的读写性能测试（未完，待整理）
dd 是 Linux/UNIX 下的一个非常有用的命令，作用是用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换

## 基本使用
测试硬盘写入（写入500个block，每个block1M）
```
time dd if=/dev/zero of=./test500M bs=1024k count=500
```
测试硬盘读取（按block1M进行读取）
```
time dd if=./test500M of=/dev/null  bs=1024k
```

## 术语解释
/dev/null，外号叫无底洞，你可以向它输出任何数据，它通吃，并且不会撑着！

/dev/zero，是一个输入设备，你可你用它来初始化文件。该设备无穷尽地提供0，可以使用任何你需要的数目——设备提供的要多的多。他可以用于向设备或文件写入字符串0。

/dev/null——它是空设备，也称为位桶（bit bucket）。任何写入它的输出都会被抛弃。如果不想让消息以标准输出显示或写入文件，那么可以将消息重定向到位桶。

## 参数说明
if=file 　　　　　　　　　　　　　　　　输入文件名，缺省为标准输入。 
of=file 　　　　　　　　　　　　　　　　输出文件名，缺省为标准输出。 
ibs=bytes 　　　　　　　　　　　　　　　一次读入 bytes 个字节(即一个块大小为 bytes 个字节)。 
obs=bytes 　　　　　　　　　　　　　　　一次写 bytes 个字节(即一个块大小为 bytes 个字节)。 
bs=bytes 　　　　　　　　　　　　　　　 同时设置读写块的大小为 bytes ，可代替 ibs 和 obs 。 
cbs=bytes 　　　　　　　　　　　　　　　一次转换 bytes 个字节，即转换缓冲区大小。 
skip=blocks 　　　　　　　　　　　　　 从输入文件开头跳过 blocks 个块后再开始复制。 
seek=blocks      　　　　　　　　　　 从输出文件开头跳过 blocks 个块后再开始复制。(通常只有当输出文件是磁盘或磁带时才有效)。 
count=blocks 　　　　　　　　　　　　　仅拷贝 blocks 个块，块大小等于 ibs 指定的字节数。 
conv=conversion[,conversion...]    用指定的参数转换文件。 
iflag=FLAGS　　　　　　　　　　　　　　指定读的方式FLAGS，参见“FLAGS参数说明”
oflag=FLAGS　　　　　　　　　　　　　　指定写的方式FLAGS，参见“FLAGS参数说明”

## 使用示例

## 注意
1. 测试读的时候，要先清除cache，如read时未清楚系统缓存，2.9GB/s，但是清除后，只有800MB/s
``` echo 3 > /proc/sys/vm/drop_caches ```
2. 测试读速度时，要加bs，不同大小速度有差异

## 疑问
1. dd和cp区别？效率是否有差异？
2. 参数默认值是多少，如bs？

## 参考
1. https://linux-mm.org/Drop_Caches
2. https://www.cnblogs.com/jiu0821/p/9854704.html
