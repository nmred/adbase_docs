# 安装 Adbase

在安装完成 gcc 、libevent 等依赖包后开始安装 Adbase, Adbase 采用 Cmake 构建项目，确保系统中已经安装了cmake

### 下载

[Download](https://github.com/weiboad/adbase/releases)

选择最新版本下来并且解压缩

### 编译安装

- cmake.sh

执行 `cmake.sh` 脚本执行编译前的预处理，生成 `build` 构建目录

```shell
[zhongxiu@bpdev adbase_v2]$ ./cmake.sh
Start cmake configure.....
编译 Debug 级别 [Debug(D)|Release(R)]: R   # D -> debug R -> release
编译环境[32bit(32)|64bit(64)]: 64
安装 adbase_kafka 模块 [Y|N]: y 
是否是开发模式 [Y|N]: n
```
	如果使用 AIMS 相关模块请安装 adbase_kafka

- 编译安装

```shell
cd build
make -j 12
make install
```
