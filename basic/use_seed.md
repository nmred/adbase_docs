# Adbase Seed 生成项目骨架代码

为了快速入门第一个项目示例采用 Adbase Seed 生成项目骨架代码，Seed 在使用时仅需要配置一个 adbase.ini 配置文件，运行 `adbase_skeleton` 命令即可。假如该项目名称为 `pattern`，生成骨架代码的步骤如下：

- 创建项目代码目录

```
mkdir pattern
cd pattern
```

- 配置 adbase.ini 

在项目根目录中创建 adbase.ini 配置文件，内容如下
```
[project]
; 项目名称
ADINF_PROJECT_NAME=pattern
; 项目描述
ADINF_PROJECT_SUMMARY=Adbase case
; 项目主页
ADINF_PROJECT_URL=https://github.com/weiboad/adbase
; 项目打包维护信息
ADINF_PROJECT_VENDOR=nmred  <nmred_2008@126.com>
ADINF_PROJECT_PACKAGER=nmred  <nmred_2008@126.com>

[module]
; 是否启用生成 adserver 相关代码，如果使用 http/memcache rpc 服务请开启
adserver=yes
; 是否启用生成 timer 相关代码，如果需要定时执行一些操作请开启
timer=no
; 是否启用生成 kafka 相关代码，如果使用 kafka 做消息队列相关操作请开启
kafkac=no
kafkap=no
; 是否启用生成 logging 相关代码，建议开启
logging=yes

[params]
; 对于 timer 模块该参数生效，定时器名称，多个定时器用逗号分隔
timers=
; 对于 adserver 模块该参数生效，http server 的 controller 名称, 多个用逗号分隔
http_controllers=Api
;对于 aims consumer 模块该参数生效，分别是kafka consumer 名称、topic、groupid, 多个用逗号分隔, 三个参数的个数必须一一对应
kafka_consumers=
kafka_consumers_topics=
kafka_consumers_groups=
;对于 aims producer 模块该参数生效，分别是kafka producer 名称、topic, 多个用逗号分隔, 两个参数的个数必须一一对应
kafka_producers=
kafka_producers_topics=

[files]
src/main.cpp=src/@ADINF_PROJECT_NAME@.cpp
rpm/main.spec.in=rpm/@ADINF_PROJECT_NAME@.spec.in

[execs]
cmake.sh=1
build_rpm.in=1
```

在这个配置文件中对代码生成比较重要的配置信息是 project 段和 module 段，project 提供给生成器项目的一些基础信息，module 用来配置生成代码模块，由于本案例中仅仅使用 RPC 服务，所以仅需要选择 adserver 和 logging 模块即可，其他模块将在后续章节详细介绍

- 运行 adbase_skeleton 生成代码

```
[zhongxiu@bpdev pattern]$ adbase_skeleton
20170220 06:21:26.441379Z 140081539787104 INFO  ./ - Seed.cpp:714
Generate file list:
	./rpm/pattern.spec.in
	./rpm/build_rpm.in
	./src/Timer.hpp
	./src/BootStrap.cpp
	./src/HeadProcessor.cpp
	./src/AdbaseConfig.hpp
	./src/McProcessor.hpp
	./src/pattern.cpp
	./src/Http.hpp
	./src/AdServer.hpp
	./src/McProcessor.cpp
	./src/HeadProcessor.hpp
	./src/AdServer.cpp
	./src/Http/Api.hpp
	./src/Http/Server.cpp
	./src/Http/HttpInterface.hpp
	./src/Http/Server.hpp
	./src/Http/Api.cpp
	./src/Http/HttpInterface.cpp
	./src/Timer.cpp
	./src/BootStrap.hpp
	./src/CMakeLists.txt
	./src/App.hpp
	./src/App.cpp
	./src/Http.cpp
	./src/Version.hpp.in
	./conf/system.ini
	./cmake.sh
	./CMakeLists.txt
```

走到这一步可能心里即窃喜又懵逼，窃喜的是很多代码自动完成了，仅仅需要写核心业务逻辑即可。懵逼的是生成这么一坨不知从何入手写业务逻辑，在正式开发前先介绍一下生成的代码的目录结构及其作用。
