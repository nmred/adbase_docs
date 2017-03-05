# 安装依赖包

Adbase 是基于 C++11 开发的所以需要 gcc 版本至少`4.8.4`, 所以对于某些版本的操作系统需要安装或者更新 gcc, 对于 Abdase的网络库实现是基于 libevent 封装的，所以 Adbase 依赖 gcc(>=4.8.4) 和 libevent, 如果公司的开发环境没有版本限制推荐使用 Centos 7.0 以上版本开发，可以免去安装升级 gcc 环境的问题

### 安装升级 gcc

本节介绍通过源码编译安装或者升级 gcc 的方式，其他方式或者具体的 Linux 衍生版见具体的安装过程。

- 下载 gcc 4.8 的源码

```shell
wget http: //ftp.gnu.org/gnu/gcc/gcc-4.8.4/gcc-4.8.4.tar.bz2
tar -jxvf  gcc-4.8.4.tar.bz2
```

- 下载编译所需依赖库

```shell
cd gcc-4.8.0
./contrib/download_prerequisites
cd ..
```

- 建立编译输出目录

```
mkdir build
```

- 生成 makefile 文件

```
cd  build
../configure --prefix=/usr/local/adinf/gcc4.8 --enable-checking=release --enable-languages=c,c++
# 如果不要32bit交叉编译环境可以加--disable-multilib 编译选项
```

- 编译安装

```
make -j 12 && make install 
```

	编译安装遇到的问题：
		报错gnu/stubs-32.h no such file or directory
		解决办法：命令 yum install glibc-devel.i686问题解决

### 安装 libevent

```
wget https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz
tar -zxf libevent-2.1.8-stable.tar.gz
cd libevent-2.1.8-stable
./configure
make -j 12
make install
```
