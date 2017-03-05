# 日期 & 时间

### 日期相关

##### 使用

日期工具类构造方法接收`(int year, int month, int day)`、`(int julianDayNum)`、`(const struct tm&)` 参数构造日期类, 支持获取年、月、日、星期、儒略日、比较日期等功能

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>
#include <time.h>

void printDate(const adbase::Date& date) {
    // get year month day
    LOG_INFO << "Year: " << date.year();
    LOG_INFO << "Month: " << date.month();
    LOG_INFO << "Day: " << date.day();
    // get week day
    LOG_INFO << "WeekDay:" << date.weekDay();
    // get julian day
    LOG_INFO << "julianDayNumber:" << date.julianDayNumber();
}

int main(void) {
    // use year month day contruct
    LOG_INFO << "Use year/month/day contruct";
    adbase::Date date1(2017, 2, 27);
    printDate(date1);

    LOG_INFO << "Use julian day contruct";
    adbase::Date date2(2457813);
    printDate(date2);

    struct tm* ptr;
    time_t rawtime;
    time(&rawtime);
    ptr = gmtime(&rawtime);
    LOG_INFO << "Use tm day contruct";
    adbase::Date date3(*ptr);
    printDate(date3);

    LOG_INFO << "Date > opt: " << (date2 > date1);
    return 0;
}
```

##### 实现细节

日期工具类通过 [儒略日](https://en.wikipedia.org/wiki/Julian_calendar) 历法计数方式实现，通过儒略日计算可以简单方便的比较日期大小或者计算日期之间的差距

关于儒略日与阳历之间转化算法见[儒略日计算器](http://www.faqs.org/faqs/calendars/faq/part2/)

### 时间相关

`Timestamp` 工具类主要作用是获取当前时间以及通过新纪元时间转化格式化时间, 使用示例：

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>
#include <time.h>

int main(void) {
    // get now time
    adbase::Timestamp time = adbase::Timestamp::now();
    LOG_INFO << time.toFormattedString();

    // get microSecondsSinceEpoch
    LOG_INFO << time.microSecondsSinceEpoch();

    time_t times = time.secondsSinceEpoch();
    LOG_INFO << times;

    time_t timet;
    ::time(&timet);
    LOG_INFO << adbase::Timestamp::fromUnixTime(timet).toFormattedString();

    adbase::Timestamp time1(1488171946600590);

    LOG_INFO << "Time diff: " << adbase::timeDifference(time, time1);

    adbase::Timestamp time3 = adbase::addTime(time, 3600);
    LOG_INFO << "Add time 3600s: " << time3.toFormattedString();
    return 0;
}
```

### 时区相关

默认情况下使用时间工具类获取到的时间是 UTC 时间格式，当需要本地时间的时候需要转化本地时间，时区工具类提供了本地时间转化 UTC时间、UTC 时间转化本地时间的方法，如下是使用方法：

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main(void) {
    adbase::TimeZone timeZone(8 * 3600, "CTS");

    adbase::Timestamp timeStamp = adbase::Timestamp::now();
    time_t seconds = timeStamp.secondsSinceEpoch();

    LOG_INFO << seconds;
    LOG_INFO << "UTC:" << timeStamp.toFormattedString();

    // convert to local time from utc
    struct tm localTime = timeZone.toLocalTime(seconds);
    // convert to time_t from struct tm
    time_t localSeconds = timeZone.fromUtcTime(localTime);
    LOG_INFO << "Local time:" << adbase::Timestamp::fromUnixTime(localSeconds).toFormattedString();

    // convert to struct tm from time_t
    struct tm convertTm = timeZone.toUtcTime(localSeconds);
    // convert to utc from local time
    time_t utcSeconds = timeZone.fromLocalTime(convertTm);
    LOG_INFO << "UTC time:" << adbase::Timestamp::fromUnixTime(utcSeconds).toFormattedString();

    return 0;
}
```
