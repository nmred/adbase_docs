# 日志

### 使用

对于后台服务开发用于错误跟踪，数据跟踪其中的一个常用的手段就是写日志，Adbase 提供了记录日志的模块，方便在开发项目的时候记录日志
 
##### 日志写入

首先需要引入 `adbase/Logging.hpp` 头文件，使用定义好的宏写日志即可, 开发者可以根据日志的等级选择对应的宏定义写日志，目前支持 `LOG_TRACE`、`LOG_DEBUG`、`LOG_INFO`、`LOG_WARN`、`LOG_ERROR`、`LOG_FATAL`、`LOG_SYSFATAL`、`LOG_SYSERR`多个日志记录级别，其中 `LOG_TRACE`、`LOG_DEBUG`、`LOG_INFO` 可以通过设置日志等级决定是否显示，其他等级的日志框架会强制记录，其中 `LOG_FATAL` 和 `LOG_SYSFATAL` 会执行 `abort` 终止程序，`LOG_FATAL` 和 `LOG_SYSFATAL` 的区别是前者一般是应用层错误是使用，后者是系统层级的错误记录，后者会记录 `errno`, 日志模块不仅可以打印常规类型，还支持 C style 格式化打印方式，使用举例：

```cpp
#include <adbase/Logging.hpp>

int main() {
    // 设置日志级别
    adbase::Logger::setLogLevel(adbase::Logger::TRACE);

    // input logging
    LOG_TRACE << "trace test";
    LOG_DEBUG << "debug test";
    LOG_INFO  << "info test";
    LOG_ERROR << "error test";

    // input pointer
    int* p;
    int a = 5;
    p = &a;
    LOG_ERROR << p;

    // 格式化的方式写入
    LOG_INFO  << adbase::Fmt("fmt %d test", 45);

    return 0;
}
```
[GITHUB 代码](https://github.com/weiboad/adbase/blob/master/example/logging_out.cpp)

##### 日志等级设置

默认是 INFO 级别，如果需要修改系统 LOG 级别可以通过两种方式修改

- 通过启动的时候设置环境变量 `ADINF_LOG_TRACE=1` 或者 `ADINF_LOG_DEBUG=1` 来设置
- 通过日志库提供的函数动态设置

```cpp
adbase::Logger::setLogLevel(adbase::Logger::TRACE);
```
[GITHUB 代码](https://github.com/weiboad/adbase/blob/master/example/logging_out.cpp#L5)

##### 日志落地

日志库在没有设置落地回调方式的情况下，默认会落地到 std::out 中，如果想落地到文件中，框架提供了一种异步高性能落地日志的实现 `adbase::AsyncLogging`, 可以通过 `adbase::Logger::setOutput()` 静态方法设置落地回调方法，如果其他落地方式该函数提供了自定义的可能。如下是通过使用 `adbase::AsyncLogging` 方式将日志落地到文件中：

```cpp
#include <adbase/Logging.hpp>
#include <signal.h>

adbase::AsyncLogging* glogger = nullptr;
void asyncLogger(const char* msg, size_t len) {
	if (glogger != nullptr) {
		glogger->append(msg, len);
	}
}
// {{{ static void killSignalMaster()

static void killSignalMaster(const int sig) {
	(void)(sig);

	LOG_ERROR << sig;
	if (glogger != nullptr) {
		LOG_INFO << "Stop...";
		delete glogger;
		glogger = nullptr;
	}

	/* 给进程组发送SIGTERM信号，结束子进程 */
	kill(0, SIGTERM);

	exit(0);
}

// }}}

int main() {
	/* 处理kill信号 */
	signal (SIGINT, killSignalMaster);
	signal (SIGKILL, killSignalMaster);
	signal (SIGQUIT, killSignalMaster);
	signal (SIGTERM, killSignalMaster);
	signal (SIGHUP, killSignalMaster);
	signal (SIGSEGV, killSignalMaster);

	// 设置日志级别
	adbase::Logger::setLogLevel(adbase::Logger::TRACE);
	LOG_TRACE << "trace test";
	LOG_DEBUG << "debug test";
	LOG_INFO  << "info test";
	LOG_ERROR << 45;
	// 格式化的方式写入
	LOG_INFO  << adbase::Fmt("%d test", 45);
	adbase::AsyncLogging*logger = new adbase::AsyncLogging("./test", 5 * 1024 * 1024);
	glogger = logger;
	adbase::Logger::setOutput(std::bind(&asyncLogger, std::placeholders::_1,  std::placeholders::_2));
	glogger->start();

	LOG_TRACE << "trace test";
	LOG_DEBUG << "debug test";
	LOG_INFO  << "info test";
	LOG_ERROR << 45;
	// 格式化的方式写入
	LOG_INFO  << adbase::Fmt("%d test", 45);

	while(true) {
		std::this_thread::sleep_for(std::chrono::seconds(1));		
	}
	return 0;
}
```
[GITHUB 代码](https://github.com/weiboad/adbase/blob/master/example/logging.cpp)
