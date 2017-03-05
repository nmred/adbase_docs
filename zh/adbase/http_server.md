# Http Server

Http Server 在实际项目比较常用，adabse 基于 libevent 实现了多线程 HTTP Server, 仅仅需要注册对应的 URL 和回调函数即可实现 HTTP 接口。Http 模块分为 Server、Request、Response 三个子模块，Server 负责 Http 交互工作，如果在C++应用程序中实现 HTTP Server 仅需要创建 Server 对象并启动即可，在启动前注册对应的 URI 及其对应回调。Request 负责请求解析，获取请求的信息可以通过 Request 获取请求信息，如请求方式、请求 URL、请求 header 等信息。Response 负责返回 HTTP 响应，返回的数据结果或者 header 可以通过该对象设置返回

简单说创建 Server 大体步骤分四步：创建 Server 容器对象、定义请求回调函数、注册请求路由、启动

### 创建 Server

创建 Server 首先要 new 一个 `adbase::http::Server` 容器对象，创建该对象需要传递  `adbase::http::Config` 配置对象，配置监听端口等信息，具体配置方法下面将介绍，创建 Server 示例代码如下：

```cpp
adbase::TimeZone tz(8*3600, "CST");
adbase::http::Config config("0.0.0.0", 10010, 3);
config.setTimeZone(tz);
config.setServerName("test");
config.setLogDir("logs");
adbase::http::Server* http = new adbase::http::Server(config);
```

#### Server 配置

##### 构造方法

在初始化 `adbase::http::Config` 对象时需要传递三个参数，分别是监听主机名、监听端口、读写超时时间，处理构造函数传递的参数配置外可以使用一下 setXxx 方式设置其他参数

```cpp
adbase::http::Config config("0.0.0.0", 10010, 3);
```

##### 参数列表

参数名 | 作用 | 举例
-|-|-
setBindAddress | 设置绑定主机名 | 0.0.0.0
setLogFormat | 日志格式化 |
setTimeZone | 设置时区 | 
setBindPort | 设置监听端口 | 10010
setTimeout | 设置读写超时时间 | 2
setLogRollSize | 设置access 日志分隔大小 | 50 \* 1024 \* 1024
setStatInterval | 设置统计运行指标的时间间隔 | 60000
setLogDir | 设置 access 日志输出目录 | logs
setServerName | 设置 server name | adbase

其中日志格式化允许开发者自定义日志输出格式，日志数据字段含义如下：


参数名 | 含义
-|-
$remote_addr | remoteAddr 
$time_local | 请求总共耗费时间 
$request_time | 请求处理时间 
$request |  请求信息 
$status |  响应状态码 
$body_bytes_sent | 响应数据大小 
$http_user_agent | UA 
$http_referer | referer 地址 

默认格式化：

```
- - $remote_addr - - - [$time_local] $request_time - "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"
```

#### 请求对象

请求对象是当 Server 接收请求后通过协议解析器解析后，将请求信息设置到请求对象中，提供给接口实现使用。该对象会在处理请求函数中作为参数传递。

请求对象中提供请求方式、URI、请求header、获取get数据、获取post 的方法, 具体参见[开发手册](https://weiboad.github.io/adbase/docs/d6/d3f/classadbase_1_1http_1_1_request.html)

#### 响应对象

响应对象和请求对象类似都是在处理请求 Server 传递到处理回调函数的对象指针。该对象提供回写响应数据的方法。如设置响应 header 、设置响应数据、响应状态的方法，具体使用参见[开发手册](https://weiboad.github.io/adbase/docs/d0/d13/classadbase_1_1http_1_1_response.html)

### 定义处理函数

请求处理函数定义如下：

```cpp
typedef std::function<void (adbase::http::Request* request,
                            adbase::http::Response* response,
							void* data)> OnRequest;
```

在 Server 中注册处理请求函数需要定义如上类型的回调函数，前两个参数分别是请求对象指针、响应对象指针，第三个参数是在注册处理函数时设置的上下文指针，供在处理请求做数据交互, 例如：

```cpp
void request(adbase::http::Request* request, adbase::http::Response* response, void* data) {
    (void)request;
    if (data != nullptr) {
        std::string* context = reinterpret_cast<std::string*>(data);
        LOG_INFO << "Context:" << *context;
    } else {
        (void)data;
    }
    response->sendReply("OK", 200, "OK");
}
```

### 注册处理函数

在定义好处理函数后，需要将其注册到 Server 中，当发生请求时会通过url 映射到对应的处理函数做处理，最终将处理的响应数据返回，注册处理函数方式如下：

```cpp
std::string test = "test string";
http->registerLocation("/debug", std::bind(request, std::placeholders::_1,
						std::placeholders::_2, std::placeholders::_3), &test);
http->registerLocation("/echo", std::bind(echo, std::placeholders::_1,
				std::placeholders::_2, std::placeholders::_3), nullptr);
```

### 总结

使用 C++ 开发http 接口需要做的就是创建 Server容器、定义处理接口函数、注册处理函数、启动这四个步骤, 如下是一个完整的例子：

```cpp
#include <adbase/Http.hpp>
#include <adbase/Logging.hpp>
#include <signal.h>

adbase::http::Server* ghttp = nullptr;
adbase::AsyncLogging* _asnclog = nullptr;

// {{{ void asyncLogger()

void asyncLogger(const char* msg, int len) {
    if (_asnclog != nullptr) {
        _asnclog->append(msg, static_cast<int>(len));
    }
}

// }}}
// {{{ void request()

void request(adbase::http::Request* request, adbase::http::Response* response, void* data) {
    (void)request;
    if (data != nullptr) {
        std::string* context = reinterpret_cast<std::string*>(data);
        LOG_INFO << "Context:" << *context;
    } else {
        (void)data;
    }
    response->sendReply("OK", 200, "OK");
}

// }}}
// {{{ void echo()

void echo(adbase::http::Request* request, adbase::http::Response* response, void* data) {
    (void)data;
    std::string post = request->getPostData();
    response->sendReply("OK", 200, post);
}

// }}}
// {{{ static void killSignal()

static void killSignal(const int sig) {
    (void)sig;
    if (ghttp != nullptr) {
        ghttp->stop();
        delete ghttp;
        ghttp = nullptr;
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
    /* 忽略Broken Pipe信号 */
    signal(SIGPIPE, SIG_IGN);
    /* 处理kill信号 */
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
    adbase::Logger::setTimeZone(tz);
    adbase::http::Config config("0.0.0.0", 10010, 3);
    config.setTimeZone(tz);
    config.setServerName("test");
    config.setLogDir("logs");

    adbase::Logger::setOutput(std::bind(asyncLogger,
                std::placeholders::_1, std::placeholders::_2));
    // 启动异步日志落地
    std::string basename = "./logs/http";
    _asnclog = new adbase::AsyncLogging(basename, 52428800);
    _asnclog->start();

    adbase::http::Server* http = new adbase::http::Server(config);
    ghttp = http;
    std::string test = "test string";
    http->registerLocation("/debug", std::bind(request, std::placeholders::_1,
                            std::placeholders::_2, std::placeholders::_3), &test);
    http->registerLocation("/echo", std::bind(echo, std::placeholders::_1,
                     std::placeholders::_2, std::placeholders::_3), nullptr);
    http->start(24);

    // do something

    while (true) {
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    }
    LOG_INFO << "exit.";
    return 0;
}
```
