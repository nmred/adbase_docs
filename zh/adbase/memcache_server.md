# Memcache Server

Adbase 实现了 Memcache 协议，可以使用 Adabse Memcache 模块编写Memcache协议的 Server, 对于 Memcache 协议可以满足的服务可以使用该模块开发，这样对于其他语言可以便捷的使用 Memcache client 交互，免去其他语言实现对应 RPC 客户端的必要，提升开发效率。较 Http 协议 Memcache 支持长连接，对于要求长连接的应用该 RPC 是自定义协议与 Http 协议一种折中选择，如果 Memcache 协议满足不了需求可以使用 Adhead 协议来完成, Adhead 定制化更强

Adbase Memcache 模块基于网络模块实现 TCP 层交互，网络IO模型使用 `adbase::net::TcpServer` 实现，该模块主要是实现交互协议，所以创建一个 Memcache Server 需要分为以下几步：

### 创建 TCP Server

```cpp
gEventLoop = new adbase::EventLoop();
adbase::InetAddress mcAddr("0.0.0.0", 10012);
adbase::TcpServer mcServer(gEventLoop->getBase(), mcAddr, "test");
```

创建 Server 首先要创建 EventLoop 管理对象，其次通过 `adbase::InetAddress` 创建地址对象，`adbase::TcpServer` 对象第一个参数是 `EventLoop` base 指针，第二个就是创建该 Server 的地址对象，第三个是 server name, 用来日志中调试使用。

### 实现协议接口

对于实现 Memcache 协议的 RPC server 实现协议接口时核心部分，实现部分仅仅需要可选的实现需要交互的命令回调函数即可，没有比较将每个一个回调函数都实现, 最终将实现对应的回调函数注册到 `adbase::mc::Interface` 中, 最终通过`adbase::mc::Handler` 实现对应 `TcpServer` 的 `ConnectionCallback` 、`CloseCallback`、`MessageCallback` 回调, 下面介绍具体的每个命令实现回调函数定义

##### Add 命令

回调函数定义格式如下：

```cpp
// AddHandler
typedef std::function<ProtocolBinaryResponseStatus (const void* key,
                                                    uint16_t keylen,
                                                    const void *data,
                                                    uint32_t datalen,
                                                    uint32_t flags,
                                                    uint32_t exptime,
                                                    uint64_t* cas)> AddHandler;
```

参数说明:

参数名 | 说明
-|-
key | key 的指针 
keylen | key 的长度 
data | 添加数据的指针 
datalen | 添加数据的长度
flags | 添加数据的参数选项
exptime | 添加数据过期时间
cas | 添加数据版本号，有 server 会返回，默认0

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::add(const void* key,
            uint16_t keylen,
            const void *data,
            uint32_t datalen,
            uint32_t flags,
            uint32_t exptime,
            uint64_t* cas) {
    (void)flags;
    (void)exptime;
    (void)cas;
    std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
    adbase::Buffer bufferBody;
    bufferBody.append(static_cast<const char*>(data), static_cast<size_t>(datalen));
    LOG_INFO << "Mc ADD key:" << keyData;
    LOG_INFO << "Mc ADD data size:" << bufferBody.readableBytes();
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

对于自定义的 Memcache 实现来说，仅仅使用 Memcache 作为数据传输交互，所以 `flags` 、`exptime` 、`cas` 参数基本不会用到可以忽略。该示例就是将客户端 add命令传输的数据通过日志打印出来

##### Append 命令

回调函数定义格式如下：

```cpp
// AppendHandler
typedef std::function<ProtocolBinaryResponseStatus (const void* key,
                                                    uint16_t keylen,
                                                    const void *data,
                                                    uint32_t datalen,
                                                    uint64_t cas,
                                                    uint64_t* resultCas)> AppendHandler;
```

参数说明:

参数名 | 说明
-|-
key | key 的指针 
keylen | key 的长度 
data | 添加数据的指针 
datalen | 添加数据的长度
cas | 上次修改数据版本号
resultCas | 本次修改数据版本号

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::append(const void* key,
            uint16_t keylen,
            const void *data,
            uint32_t datalen,
            uint64_t cas,
            uint64_t* resultCas) {
    (void)cas;
    (void)resultCas;
    std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
    adbase::Buffer bufferBody;
    bufferBody.append(static_cast<const char*>(data), static_cast<size_t>(datalen));
    LOG_INFO << "Mc APPEND key:" << keyData;
    LOG_INFO << "Mc APPEND data size:" << bufferBody.readableBytes();
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

对于自定义的 Memcache 实现来说，仅仅使用 Memcache 作为数据传输交互，所以 `cas` 、`resultCas` 参数基本不会用到可以忽略。该示例就是将客户端 `append` 命令传输的数据通过日志打印出来

##### Decrement 命令

回调函数定义格式如下：

```cpp
// DecrementHandler
typedef std::function<ProtocolBinaryResponseStatus (const void* key,
                                                    uint16_t keylen,
                                                    uint64_t delta,
                                                    uint64_t initial,
                                                    uint32_t expiration,
                                                    uint64_t* result,
                                                    uint64_t* resultCas)> DecrementHandler;
```

参数说明:

参数名 | 说明
-|-
key | key 的指针 
keylen | key 的长度 
delta | 修改数据的 offset 
initial | 如果不存在默认值
expiration | 有效期
result | 变更后的结果数据
resultCas | 本次修改数据版本号

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::decrement(const void* key,
            uint16_t keylen,
            uint64_t delta,
            uint64_t initial,
            uint32_t expiration,
            uint64_t* result,
            uint64_t* resultCas) {
    (void)delta;
    (void)initial;
    (void)expiration;
    (void)result;
    (void)resultCas;
    std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
    LOG_INFO << "Mc DECREMENT key:" << keyData;
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

对于自定义的 Memcache 实现来说，仅仅使用 Memcache 作为数据传输交互，所以 `cas` 、`resultCas` 参数基本不会用到可以忽略。该示例就是将客户端 `decrement` 命令传输的数据key通过日志打印出来

##### Delete 命令

回调函数定义格式如下：

```cpp
// DeleteHandler
typedef std::function<ProtocolBinaryResponseStatus (const void* key,
                                                    uint16_t keylen,
                                                    uint64_t cas)> DeleteHandler;
```

参数说明:

参数名 | 说明
-|-
key | key 的指针 
keylen | key 的长度 

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::deleteOp(const void* key,
            uint16_t keylen,
            uint64_t cas) {
    (void)cas;
    std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
    LOG_INFO << "Mc DELETE key:" << keyData;
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

该示例就是将客户端 `delete` 命令传输的数据 `key` 通过日志打印出来

##### Flush 命令

回调函数定义格式如下：

```cpp
// FlushHandler
typedef std::function<ProtocolBinaryResponseStatus (uint32_t when)> FlushHandler;
```

参数说明:

参数名 | 说明
-|-
when | 将前几秒的数据 flush进去

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::flush(uint32_t when) {
    LOG_INFO << "Mc FLUSH when:" << when;
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```
##### Get 命令

回调函数定义格式如下：

```cpp
// GetHandler
typedef std::function<ProtocolBinaryResponseStatus (const void* key,
                                                    uint16_t keylen,
                                                    Buffer *data)> GetHandler;
```

参数说明:

参数名 | 说明
-|-
key | key 的指针 
keylen | key 的长度 
data |  get 操作将要返回的数据 

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::get(const void* key,
            uint16_t keylen,
            adbase::Buffer *data) {
    std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
    data->append(static_cast<const char*>(key), static_cast<size_t>(keylen));
    LOG_INFO << "Mc GET key:" << keyData;
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

该示例就是将客户端 `get` 命令传输的数据 `key` 通过日志打印出来, 并且将 `key` 返回给客户端

##### Increment 命令

回调函数定义格式如下：

```cpp
// IncrementHandler
typedef std::function<ProtocolBinaryResponseStatus (const void* key,
                                                    uint16_t keylen,
                                                    uint64_t delta,
                                                    uint64_t initial,
                                                    uint32_t expiration,
                                                    uint64_t* result,
                                                    uint64_t* resultCas)> IncrementHandler;
```

参数说明:

参数名 | 说明
-|-
key | key 的指针 
keylen | key 的长度 
delta | 修改数据的 offset 
initial | 如果不存在默认值
expiration | 有效期
result | 变更后的结果数据
resultCas | 本次修改数据版本号

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::increment(const void* key,
            uint16_t keylen,
            uint64_t delta,
            uint64_t initial,
            uint32_t expiration,
            uint64_t* result,
            uint64_t* resultCas) {
    (void)delta;
    (void)initial;
    (void)expiration;
    (void)result;
    (void)resultCas;
    std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
    LOG_INFO << "Mc INCREMENT key:" << keyData;
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

对于自定义的 Memcache 实现来说，仅仅使用 Memcache 作为数据传输交互，所以 `cas` 、`resultCas` 参数基本不会用到可以忽略。该示例就是将客户端 `increment` 命令传输的数据key通过日志打印出来

##### Prepend 命令

回调函数定义格式如下：

```cpp
// PrependHandler
typedef std::function<ProtocolBinaryResponseStatus (const void* key,
                                                    uint16_t keylen,
                                                    const void *data,
                                                    uint32_t datalen,
                                                    uint64_t cas,
                                                    uint64_t* resultCas)> PrependHandler;
```

参数说明:

参数名 | 说明
-|-
key | key 的指针 
keylen | key 的长度 
data | 添加数据的指针 
datalen | 添加数据的长度
cas | 上次修改数据版本号
resultCas | 本次修改数据版本号

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::prepend(const void* key,
            uint16_t keylen,
            const void *data,
            uint32_t datalen,
            uint64_t cas,
            uint64_t* resultCas) {
    (void)cas;
    (void)resultCas;
    std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
    adbase::Buffer bufferBody;
    bufferBody.append(static_cast<const char*>(data), static_cast<size_t>(datalen));
    LOG_INFO << "Mc PREPEND key:" << keyData;
    LOG_INFO << "Mc PREPEND data size:" << bufferBody.readableBytes();
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

对于自定义的 Memcache 实现来说，仅仅使用 Memcache 作为数据传输交互，所以 `cas` 、`resultCas` 参数基本不会用到可以忽略。该示例就是将客户端 `prepend` 命令传输的数据通过日志打印出来

##### Quit 命令

回调函数定义格式如下：

```cpp
// QuitHandler
typedef std::function<ProtocolBinaryResponseStatus ()> QuitHandler;
```

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::quit() {
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

该示例是在客户端执行 `quit` 命令时记录日志

##### replace 命令

回调函数定义格式如下：

```cpp
// ReplaceHander
typedef std::function<ProtocolBinaryResponseStatus (const void* key,
                                                    uint16_t keylen,
                                                    const void *data,
                                                    uint32_t datalen,
                                                    uint32_t flags,
                                                    uint32_t exptime,
                                                    uint64_t cas,
                                                    uint64_t* resultCas)> ReplaceHander;
```

参数说明:

参数名 | 说明
-|-
key | key 的指针 
keylen | key 的长度 
data | 添加数据的指针 
datalen | 添加数据的长度
flags | 添加数据的参数选项
exptime | 添加数据过期时间
cas | 上次修改数据版本号
resultCas | 本次修改数据版本号

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::replace(const void* key,
            uint16_t keylen,
            const void *data,
            uint32_t datalen,
            uint32_t flags,
            uint32_t exptime,
            uint64_t cas,
            uint64_t* resultCas) {
    (void)flags;
    (void)exptime;
    (void)cas;
    (void)resultCas;
    std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
    adbase::Buffer bufferBody;
    bufferBody.append(static_cast<const char*>(data), static_cast<size_t>(datalen));
    LOG_INFO << "Mc REPLACE key:" << keyData;
    LOG_INFO << "Mc REPLACE data size:" << bufferBody.readableBytes();
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

对于自定义的 Memcache 实现来说，仅仅使用 Memcache 作为数据传输交互，所以 `flags`、`exptime`、`cas` 、`resultCas` 参数基本不会用到可以忽略。该示例就是将客户端 `replace` 命令传输的数据通过日志打印出来

##### Set 命令

回调函数定义格式如下：

```cpp
// SetHandler
typedef std::function<ProtocolBinaryResponseStatus (const void* key,
                                                    uint16_t keylen,
                                                    const void *data,
                                                    uint32_t datalen,
                                                    uint32_t flags,
                                                    uint32_t exptime,
                                                    uint64_t cas,
                                                    uint64_t* resultCas)> SetHandler;
```

参数说明:

参数名 | 说明
-|-
key | key 的指针 
keylen | key 的长度 
data | 添加数据的指针 
datalen | 添加数据的长度
flags | 添加数据的参数选项
exptime | 添加数据过期时间
cas | 上次修改数据版本号
resultCas | 本次修改数据版本号

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::set(const void* key,
            uint16_t keylen,
            const void *data,
            uint32_t datalen,
            uint32_t flags,
            uint32_t exptime,
            uint64_t cas,
            uint64_t* resultCas) {
    (void)flags;
    (void)exptime;
    (void)cas;
    (void)resultCas;
    std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
    adbase::Buffer bufferBody;
    bufferBody.append(static_cast<const char*>(data), static_cast<size_t>(datalen));
    LOG_INFO << "Mc SET key:" << keyData;
    LOG_INFO << "Mc SET data size:" << bufferBody.readableBytes();
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

对于自定义的 Memcache 实现来说，仅仅使用 Memcache 作为数据传输交互，所以 `flags`、`exptime`、`cas` 、`resultCas` 参数基本不会用到可以忽略。该示例就是将客户端 `set` 命令传输的数据通过日志打印出来

##### Stat 命令

回调函数定义格式如下：

```cpp
// StatHandler
typedef std::function<ProtocolBinaryResponseStatus (const void* key,
                                                    uint16_t keylen,
                                                    Buffer *data)> StatHandler;
```

参数说明:

参数名 | 说明
-|-
key | key 的指针 
keylen | key 的长度 
data |  stat 操作将要返回的数据 

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::stat(const void* key,
            uint16_t keylen,
            adbase::Buffer *data) {
    std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
    data->append(static_cast<const char*>(key), static_cast<size_t>(keylen));
    LOG_INFO << "Mc STAT key:" << keyData;
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

该示例就是将客户端 `stat` 命令传输的数据 `key` 通过日志打印出来, 并且将 `key` 返回给客户端

##### Version 命令

回调函数定义格式如下：

```cpp
// VersionHandler
typedef std::function<ProtocolBinaryResponseStatus (Buffer *data)> VersionHandler;
```

参数说明:

参数名 | 说明
-|-
data |  version 操作将要返回的数据 

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::version(adbase::Buffer *data) {
    data->append("0.1.0");
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

该示例就是返回 server 版本 `0.1.0`

#####  Verbosity 命令

回调函数定义格式如下：

```cpp
// VerbosityHandler
typedef std::function<ProtocolBinaryResponseStatus (uint32_t verbosity)> VerbosityHandler;
```

参数说明:

参数名 | 说明
-|-
verbosity | uint32_t

Example:

```cpp
adbase::mc::ProtocolBinaryResponseStatus McProcessor::verbosity(uint32_t verbose) {
    LOG_INFO << "Mc FLUSH verbose:" << verbose;
    return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
}
```

##### 命令执行前调用回调

回调函数定义格式如下：

```cpp
// PreExecute
typedef std::function<void ()> PreExecute;
```

当 server 接收到命令时将先执行 `PreExecute` 回调，再执行对应的命令回调


##### 命令执行后调用回调

回调函数定义格式如下：

```cpp
// PostExecute
typedef std::function<void ()> PostExecute;
```

当 server 处理完对应的命令时将执行 `PostExecute` 回调, 再返回

##### 命令解析错误

回调函数定义格式如下：

```cpp
// UnknownExecute
typedef std::function<void ()> UnknownExecute;
```

当命令解析错误的时候会执行该回调函数

### 注册 TCP Server 回调 

当定义实现了 Memcache 接口回调后需要创建 `adbase::mc::Interface` 接口容器对象，并且将要实现的命令回调函数设置进去，通过 `adbase::mc::Interface` 指针构建 `adbase::mc::Handler`, `adbase::mc::Handler` 的作用就是该对象自动实现了 TcpServer 需要的三个回调函数实现，创建出来该对象就可以直接绑定，如下是从创建 TcpServer 到注册回调完成的完整代码：

```cpp
// McProcessor
class McProcessor {
public:
	McProcessor() {}
	~McProcessor() {}

	adbase::mc::ProtocolBinaryResponseStatus set(const void* key,
			uint16_t keylen,
			const void *data,
			uint32_t datalen,
			uint32_t flags,
			uint32_t exptime,
			uint64_t cas,
			uint64_t* resultCas) {
		(void)flags;
		(void)exptime;
		(void)cas;
		(void)resultCas;
		//std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
		//adbase::Buffer bufferBody;
		//bufferBody.append(static_cast<const char*>(data), static_cast<size_t>(datalen));
		//LOG_INFO << "Mc SET key:" << keyData;
		//LOG_INFO << "Mc SET data size:" << bufferBody.readableBytes();
		return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
	}

	adbase::mc::ProtocolBinaryResponseStatus get(const void* key,
			uint16_t keylen,
			adbase::Buffer *data) {
		//std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
		data->append(static_cast<const char*>(key), static_cast<size_t>(keylen));
		//LOG_INFO << "Mc GET key:" << keyData;
		return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
	}
};
```
```cpp
gEventLoop = new adbase::EventLoop();

McProcessor mcProcessor;
adbase::InetAddress mcAddr("0.0.0.0", 10012);
adbase::TcpServer mcServer(gEventLoop->getBase(), mcAddr, "test");
adbase::mc::Interface mcInterface;

mcInterface.setSetHandler(std::bind(&McProcessor::set, &mcProcessor, std::placeholders::_1,
			std::placeholders::_2, std::placeholders::_3,
			std::placeholders::_4, std::placeholders::_5,
			std::placeholders::_6, std::placeholders::_7,
			std::placeholders::_8));
mcInterface.setGetHandler(std::bind(&McProcessor::get, &mcProcessor, std::placeholders::_1,
			std::placeholders::_2, std::placeholders::_3));
adbase::mc::Handler mcHandler(&mcInterface);
mcServer.setConnectionCallback(std::bind(&adbase::mc::Handler::onConnection, &mcHandler,
			std::placeholders::_1));
mcServer.setCloseCallback(std::bind(&adbase::mc::Handler::onClose, &mcHandler, std::placeholders::_1));
mcServer.setMessageCallback(std::bind(&adbase::mc::Handler::onMessage, &mcHandler, std::placeholders::_1,
			std::placeholders::_2, std::placeholders::_3));
mcServer.start(24);

gEventLoop->start();
```
### 启动

启动 server 直接执行 `TcpServer->start` 方法即可，start 参数是该 server 的 worker 线程数, 如下是完成的代码：

```cpp
#include <adbase/Logging.hpp>
#include <adbase/Net.hpp>
#include <adbase/Mc.hpp>
#include <signal.h>

adbase::EventLoop* gEventLoop = nullptr;
adbase::AsyncLogging* _asnclog = nullptr;

// {{{ class McProcessor

class McProcessor {
public:
	McProcessor() {}
	~McProcessor() {}

	adbase::mc::ProtocolBinaryResponseStatus set(const void* key,
			uint16_t keylen,
			const void *data,
			uint32_t datalen,
			uint32_t flags,
			uint32_t exptime,
			uint64_t cas,
			uint64_t* resultCas) {
		(void)flags;
		(void)exptime;
		(void)cas;
		(void)resultCas;
		//std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
		//adbase::Buffer bufferBody;
		//bufferBody.append(static_cast<const char*>(data), static_cast<size_t>(datalen));
		//LOG_INFO << "Mc SET key:" << keyData;
		//LOG_INFO << "Mc SET data size:" << bufferBody.readableBytes();
		return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
	}

	adbase::mc::ProtocolBinaryResponseStatus get(const void* key,
			uint16_t keylen,
			adbase::Buffer *data) {
		//std::string keyData(static_cast<const char*>(key), static_cast<size_t>(keylen));
		data->append(static_cast<const char*>(key), static_cast<size_t>(keylen));
		//LOG_INFO << "Mc GET key:" << keyData;
		return adbase::mc::PROTOCOL_BINARY_RESPONSE_SUCCESS;
	}
};
// }}}
// {{{ void asyncLogger()

void asyncLogger(const char* msg, int len) {
	if (_asnclog != nullptr) {
		_asnclog->append(msg, static_cast<int>(len));
	}
}

// }}}
// {{{ static void killSignal()

static void killSignal(const int sig) {
	(void)sig;
	if (gEventLoop != nullptr) {
		gEventLoop->stop();
		delete gEventLoop;
	}
	exit(0);
}

// }}}
// {{{ static void reloadConf()

static void reloadConf(const int sig) {
	(void)sig;
}

// }}}
// {{{ static void registerSignal()

static void registerSignal() {
	/// 忽略Broken Pipe信号
	signal(SIGPIPE, SIG_IGN);
	/// 处理kill信号
	signal(SIGINT,  killSignal);
	signal(SIGKILL, killSignal);
	signal(SIGQUIT, killSignal);
	signal(SIGTERM, killSignal);
	signal(SIGHUP,  killSignal);
	signal(SIGSEGV, killSignal);
	signal(SIGUSR1, reloadConf);
}

// }}}

int main(void) {
	registerSignal();
	adbase::TimeZone tz(8*3600, "CST");

	adbase::Logger::setOutput(std::bind(asyncLogger,
				std::placeholders::_1, std::placeholders::_2));
	// 启动异步日志落地
	std::string basename = "./logs/mc";
	_asnclog = new adbase::AsyncLogging(basename, 52428800);
	_asnclog->start();

	adbase::Logger::setTimeZone(tz);
	gEventLoop = new adbase::EventLoop();

	McProcessor mcProcessor;
	adbase::InetAddress mcAddr("0.0.0.0", 10012);
	adbase::TcpServer mcServer(gEventLoop->getBase(), mcAddr, "test");
	adbase::mc::Interface mcInterface;

	mcInterface.setSetHandler(std::bind(&McProcessor::set, &mcProcessor, std::placeholders::_1,
				std::placeholders::_2, std::placeholders::_3,
				std::placeholders::_4, std::placeholders::_5,
				std::placeholders::_6, std::placeholders::_7,
				std::placeholders::_8));
	mcInterface.setGetHandler(std::bind(&McProcessor::get, &mcProcessor, std::placeholders::_1,
				std::placeholders::_2, std::placeholders::_3));
	adbase::mc::Handler mcHandler(&mcInterface);
	mcServer.setConnectionCallback(std::bind(&adbase::mc::Handler::onConnection, &mcHandler,
				std::placeholders::_1));
	mcServer.setCloseCallback(std::bind(&adbase::mc::Handler::onClose, &mcHandler, std::placeholders::_1));
	mcServer.setMessageCallback(std::bind(&adbase::mc::Handler::onMessage, &mcHandler, std::placeholders::_1,
				std::placeholders::_2, std::placeholders::_3));
	mcServer.start(24);

	gEventLoop->start();

	return 0;
}
```
