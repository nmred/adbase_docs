# 度量工具库

度量工具库通过在C++代码中嵌入进行度量运行时的各项指标，该工具库提供 Gauges、Meters、Counter、Histograms、Timers 五类常用的通用统计指标方式, 使用的方式是在程序入口开启指标统计，在程序中嵌入度量代码，通过 `adbase::metrics::Metrics->getXxx` 方法获取最终的统计结果, 如下是介绍每个类型的度量工具使用方法

### Gauges

Gauges 度量方式是度量间隔时间的某个指标的瞬时值，比如要度量当前程序中某个 HashMap 的大小就可以用该度量工具统计，使用方法是获取 Gauges 实例，通过注册统计时间间隔和统计回调函数的方式度量指标

```cpp
// init metrics
adbase::metrics::Metrics::init(&timer);

// register gauges
adbase::metrics::Metrics::buildGauges("test", "rss", interval, [](){
	return 100;
});
```

如上代码中所示要使用 gauges 类型的度量，首选要初始化整个度量工具库，其中参数是一个定时器。下一步是注册gauges 回调函数及其指标的模块名和指标名称

如下是一个完整的示例，统计 `test` 模块下的 `rss` 指标每隔 1s

```cpp
#include <signal.h>
#include <adbase/Net.hpp>
#include <adbase/Logging.hpp>
#include <adbase/Metrics.hpp>

adbase::metrics::Metrics* metrics = nullptr;
adbase::EventLoop* gloop = nullptr;

void test(void*) {
    if (metrics != nullptr) {
        std::unordered_map<std::string, int64_t> values = metrics->getGauges();
        for (auto &t : values) {
            LOG_INFO << t.first << " -> " << t.second;
        }
    }
}

// {{{ static void killSignal()

static void killSignal(const int sig) {
    (void)sig;
    if (gloop != nullptr) {
        gloop->stop();
        delete gloop;
        gloop = nullptr;
    }

    adbase::metrics::Metrics::stop();
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
    adbase::EventLoop* eventloop = new adbase::EventLoop();
    gloop = eventloop;
    adbase::Timer timer(eventloop->getBase());
    metrics = adbase::metrics::Metrics::init(&timer);

    uint32_t interval = 1000;
    timer.runEvery(interval, std::bind(test, std::placeholders::_1), nullptr);
    adbase::metrics::Metrics::buildGauges("test", "rss", interval, [](){
        return 100;
    });
    eventloop->start();
    return 0;
}
```

运行结果：

```
20170302 02:44:58.554947Z 140376105634144 INFO  testrss -> 100 - gauges.cpp:13
20170302 02:44:59.555897Z 140376105634144 INFO  testrss -> 100 - gauges.cpp:13
20170302 02:45:00.556301Z 140376105634144 INFO  testrss -> 100 - gauges.cpp:13
20170302 02:45:01.556691Z 140376105634144 INFO  testrss -> 100 - gauges.cpp:13
20170302 02:45:02.556808Z 140376105634144 INFO  testrss -> 100 - gauges.cpp:13
20170302 02:45:03.557219Z 140376105634144 INFO  testrss -> 100 - gauges.cpp:13
```

### Counter

Counter 是用来计数累加值得度量类型，提供了 `add` 、`dec` 方法进行对指标进行累加计算, 比如统计当前请求总量可以用该类型度量工具

使用方法和 Gauges 类似，也是首先要初始化，然后创建 Counter 对象，使用 Counter 对象对指标进行计算累加，例如要统计请求量如下：

```cpp
#include <signal.h>
#include <adbase/Net.hpp>
#include <adbase/Logging.hpp>
#include <adbase/Metrics.hpp>

adbase::metrics::Metrics* metrics = nullptr;
adbase::EventLoop* gloop = nullptr;

void test(void*) {
    if (metrics != nullptr) {
        std::unordered_map<std::string, int64_t> values = metrics->getCounter();
        for (auto &t : values) {
            LOG_INFO << t.first << " -> " << t.second;
        }
    }
}

// {{{ static void killSignal()

static void killSignal(const int sig) {
    (void)sig;
    if (gloop != nullptr) {
        gloop->stop();
        delete gloop;
        gloop = nullptr;
    }

    adbase::metrics::Metrics::stop();
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
    adbase::EventLoop* eventloop = new adbase::EventLoop();
    gloop = eventloop;
    adbase::Timer timer(eventloop->getBase());
    //metrics = adbase::metrics::Metrics::init(&timer);

    uint32_t interval = 1000;
    timer.runEvery(interval, std::bind(test, std::placeholders::_1), nullptr);
    adbase::metrics::Counter* counter = adbase::metrics::Metrics::buildCounter("test", "request");
    if (counter != nullptr) {
        counter->add(9);
        counter->dec(3);
    }
    eventloop->start();
    return 0;
}
```

运行结果：

```
20170302 02:46:11.321117Z 140036159633760 INFO  testrequest -> 6 - counter.cpp:13
20170302 02:46:12.321437Z 140036159633760 INFO  testrequest -> 6 - counter.cpp:13
20170302 02:46:13.322288Z 140036159633760 INFO  testrequest -> 6 - counter.cpp:13
20170302 02:46:14.322580Z 140036159633760 INFO  testrequest -> 6 - counter.cpp:13
20170302 02:46:15.322904Z 140036159633760 INFO  testrequest -> 6 - counter.cpp:13
20170302 02:46:16.323421Z 140036159633760 INFO  testrequest -> 6 - counter.cpp:13
20170302 02:46:17.323820Z 140036159633760 INFO  testrequest -> 6 - counter.cpp:13
```

### Meters

Meters 一般是用来度量某个指标的速率的，例如统计当前系统的load 或者当前应用程序的请求 QPS, 该度量工具会自动计算出 1min、5min、15min 的平均速度,例如

```cpp
#include <signal.h>
#include <adbase/Net.hpp>
#include <adbase/Logging.hpp>
#include <adbase/Metrics.hpp>

adbase::metrics::Metrics* metrics = nullptr;
adbase::metrics::Meters* meters = nullptr;
adbase::EventLoop* gloop = nullptr;

void test(void*) {
    if (metrics != nullptr) {
        std::unordered_map<std::string, adbase::metrics::MeterItem> values = metrics->getMeters();
        for (auto &t : values) {
            LOG_INFO << t.first << " : ";
            LOG_INFO << "\tcount     = " << t.second.count;
            LOG_INFO << "\tmean rate = " << t.second.meanRate;
            LOG_INFO << "\t1-minute rate = " << t.second.min1Rate;
            LOG_INFO << "\t5-minute rate = " << t.second.min5Rate;
            LOG_INFO << "\t15-minute rate = " << t.second.min15Rate;
        }
    }

    int num = rand() % 1000;
    for (int i = 0; i < num; i++) {
        if (meters != nullptr) {
            meters->mark();
        }
    }
}

// {{{ static void killSignal()

static void killSignal(const int sig) {
    (void)sig;
    if (gloop != nullptr) {
        gloop->stop();
        delete gloop;
        gloop = nullptr;
    }

    adbase::metrics::Metrics::stop();
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
    adbase::EventLoop* eventloop = new adbase::EventLoop();
    gloop = eventloop;
    adbase::Timer timer(eventloop->getBase());
    metrics = adbase::metrics::Metrics::init(&timer);

    uint32_t interval = 1000;
    timer.runEvery(interval, std::bind(test, std::placeholders::_1), nullptr);
    meters = adbase::metrics::Metrics::buildMeters("test", "request");

    eventloop->start();
    return 0;
}
```

运行结果：

```
20170302 02:53:51.492440Z 140362442348896 INFO  testrequest :  - meters.cpp:14
20170302 02:53:51.492472Z 140362442348896 INFO  	count     = 96405 - meters.cpp:15
20170302 02:53:51.492482Z 140362442348896 INFO  	mean rate = 494.384 - meters.cpp:16
20170302 02:53:51.492495Z 140362442348896 INFO  	1-minute rate = 542.433 - meters.cpp:17
20170302 02:53:51.492506Z 140362442348896 INFO  	5-minute rate = 0 - meters.cpp:18
20170302 02:53:51.492516Z 140362442348896 INFO  	15-minute rate = 0 - meters.cpp:19
```

### Histograms

直方图度量工具用来统计某个指标分布情况，例如要统计某个接口的请求的响应时间的分布状况，使用该工具可以计算分布百分比对应的值, 使用 Histograms 需要获取 Histograms 实例对象，获取方法如下：

```cpp
histograms = adbase::metrics::Metrics::buildHistograms("test", "request", interval);
```

示例代码：

```cpp
#include <signal.h>
#include <adbase/Net.hpp>
#include <adbase/Logging.hpp>
#include <adbase/Metrics.hpp>

adbase::metrics::Metrics* metrics = nullptr;
adbase::metrics::Histograms* histograms = nullptr;
adbase::EventLoop* gloop = nullptr;

void test(void*) {
    if (metrics != nullptr) {
        std::unordered_map<std::string, adbase::metrics::HistogramsItem> values = metrics->getHistograms();
        for (auto &t : values) {
            LOG_INFO << t.first << " : ";
            LOG_INFO << "\tmin    = " << t.second.min;
            LOG_INFO << "\tmax    = " << t.second.max;
            LOG_INFO << "\tmean   = " << t.second.mean;
            LOG_INFO << "\tstddev = " << t.second.stddev;
            LOG_INFO << "\tmedian = " << t.second.median;
            LOG_INFO << "\t75%   <= " << t.second.percent75;
            LOG_INFO << "\t95%   <= " << t.second.percent95;
            LOG_INFO << "\t98%   <= " << t.second.percent98;
            LOG_INFO << "\t99%   <= " << t.second.percent99;
            LOG_INFO << "\t99.9%   <= " << t.second.percent999;
        }
    }

    for (int i = 0; i < 10000; i++) {
        if (histograms != nullptr) {
            histograms->update(rand() % 1000);
        }
    }
}

// {{{ static void killSignal()

static void killSignal(const int sig) {
    (void)sig;
    if (gloop != nullptr) {
        gloop->stop();
        delete gloop;
        gloop = nullptr;
    }

    adbase::metrics::Metrics::stop();
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
    adbase::EventLoop* eventloop = new adbase::EventLoop();
    gloop = eventloop;
    adbase::Timer timer(eventloop->getBase());
    metrics = adbase::metrics::Metrics::init(&timer);

    uint32_t interval = 1000;
    timer.runEvery(interval, std::bind(test, std::placeholders::_1), nullptr);
    histograms = adbase::metrics::Metrics::buildHistograms("test", "request", interval);

    eventloop->start();
    return 0;
}
```

运行结果:

```cpp
20170302 02:59:33.267863Z 139981904234848 INFO  testrequest :  - histograms.cpp:14
20170302 02:59:33.267901Z 139981904234848 INFO  	min    = 0 - histograms.cpp:15
20170302 02:59:33.267915Z 139981904234848 INFO  	max    = 999 - histograms.cpp:16
20170302 02:59:33.267935Z 139981904234848 INFO  	mean   = 74.147 - histograms.cpp:17
20170302 02:59:33.267947Z 139981904234848 INFO  	stddev = 580.814 - histograms.cpp:18
20170302 02:59:33.267958Z 139981904234848 INFO  	median = 506 - histograms.cpp:19
20170302 02:59:33.267968Z 139981904234848 INFO  	75%   <= 755 - histograms.cpp:20
20170302 02:59:33.267978Z 139981904234848 INFO  	95%   <= 952 - histograms.cpp:21
20170302 02:59:33.267995Z 139981904234848 INFO  	98%   <= 980 - histograms.cpp:22
20170302 02:59:33.268005Z 139981904234848 INFO  	99%   <= 990 - histograms.cpp:23
20170302 02:59:33.268016Z 139981904234848 INFO  	99.9%   <= 999 - histograms.cpp:24
```

### Timers

此处的 Timers 不是网络模块中定时器，而是一种度量方式，可以理解为是 Histograms 和 Meters 的结合体, 统计计算的数据同时具有 Histograms 和 Meters 的参数指标, 使用方法如下：

```cpp
// get timers object
timers = adbase::metrics::Metrics::buildTimers("test", "request", interval);

adbase::metrics::Timer timer;
timer.start();
// do something
std::this_thread::sleep_for(std::chrono::milliseconds(rand() % 2));
timers->setTimer(timer.stop());
```

实例代码：

```cpp
#include <signal.h>
#include <adbase/Net.hpp>
#include <adbase/Logging.hpp>
#include <adbase/Metrics.hpp>

adbase::metrics::Metrics* metrics = nullptr;
adbase::metrics::Timers* timers = nullptr;
adbase::EventLoop* gloop = nullptr;

void test1(void*) {
    while(true) {
        int num = rand() % 10000;
        for (int i = 0; i < num; i++) {
            if (timers != nullptr) {
                adbase::metrics::Timer timer;
                timer.start();
                std::this_thread::sleep_for(std::chrono::milliseconds(rand() % 2));
                timers->setTimer(timer.stop());
            }
        }
    }
}
void test(void*) {
    if (metrics != nullptr) {
        std::unordered_map<std::string, adbase::metrics::TimersItem> values = metrics->getTimers();
        for (auto &t : values) {
            adbase::metrics::MetricName name;
            name = adbase::metrics::Metrics::getMetricName(t.first);
            LOG_INFO << name.moduleName << "." << name.metricName << " : ";
            LOG_INFO << "\tcount     = " << t.second.meter.count;
            LOG_INFO << "\tmean rate = " << t.second.meter.meanRate;
            LOG_INFO << "\t1-minute rate = " << t.second.meter.min1Rate;
            LOG_INFO << "\t5-minute rate = " << t.second.meter.min5Rate;
            LOG_INFO << "\t15-minute rate = " << t.second.meter.min15Rate;
            LOG_INFO << "\tmin    = " << t.second.histogram.min;
            LOG_INFO << "\tmax    = " << t.second.histogram.max;
            LOG_INFO << "\tmean   = " << t.second.histogram.mean;
            LOG_INFO << "\tstddev = " << t.second.histogram.stddev;
            LOG_INFO << "\tmedian = " << t.second.histogram.median;
            LOG_INFO << "\t75%   <= " << t.second.histogram.percent75;
            LOG_INFO << "\t95%   <= " << t.second.histogram.percent95;
            LOG_INFO << "\t98%   <= " << t.second.histogram.percent98;
            LOG_INFO << "\t99%   <= " << t.second.histogram.percent99;
            LOG_INFO << "\t99.9%   <= " << t.second.histogram.percent999;
        }
    }
}

// {{{ static void killSignal()

static void killSignal(const int sig) {
    (void)sig;
    if (gloop != nullptr) {
        gloop->stop();
        delete gloop;
        gloop = nullptr;
    }

    adbase::metrics::Metrics::stop();
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
    adbase::EventLoop* eventloop = new adbase::EventLoop();
    gloop = eventloop;
    adbase::Timer timer(eventloop->getBase());
    metrics = adbase::metrics::Metrics::init(&timer);

    uint32_t interval = 1000;
    timer.runEvery(interval, std::bind(test, std::placeholders::_1), nullptr);
    timers = adbase::metrics::Metrics::buildTimers("test", "request", interval);

    std::thread* t = new std::thread(std::bind(test1, std::placeholders::_1), nullptr);

    eventloop->start();
    t->join();
    return 0;
}
```

运行结果：

```
20170302 03:03:24.776028Z 139685675747680 INFO  test.request :  - timers.cpp:29
20170302 03:03:24.776065Z 139685675747680 INFO  	count     = 7569 - timers.cpp:30
20170302 03:03:24.776076Z 139685675747680 INFO  	mean rate = 1513.8 - timers.cpp:31
20170302 03:03:24.776090Z 139685675747680 INFO  	1-minute rate = 0 - timers.cpp:32
20170302 03:03:24.776100Z 139685675747680 INFO  	5-minute rate = 0 - timers.cpp:33
20170302 03:03:24.776110Z 139685675747680 INFO  	15-minute rate = 0 - timers.cpp:34
20170302 03:03:24.776120Z 139685675747680 INFO  	min    = 0 - timers.cpp:35
20170302 03:03:24.776130Z 139685675747680 INFO  	max    = 1.176 - timers.cpp:36
20170302 03:03:24.776140Z 139685675747680 INFO  	mean   = 0.506 - timers.cpp:37
20170302 03:03:24.776150Z 139685675747680 INFO  	stddev = 0.733 - timers.cpp:38
20170302 03:03:24.776160Z 139685675747680 INFO  	median = 0.001 - timers.cpp:39
20170302 03:03:24.776170Z 139685675747680 INFO  	75%   <= 1.06 - timers.cpp:40
20170302 03:03:24.776180Z 139685675747680 INFO  	95%   <= 1.083 - timers.cpp:41
20170302 03:03:24.776190Z 139685675747680 INFO  	98%   <= 1.094 - timers.cpp:42
20170302 03:03:24.776200Z 139685675747680 INFO  	99%   <= 1.099 - timers.cpp:43
20170302 03:03:24.776209Z 139685675747680 INFO  	99.9%   <= 1.138 - timers.cpp:44
```
