# 序列号生成

序列号工具类是为了生成唯一序列号的，序列号由 uint64 类型存储，支持由 `base62` 算法转化成字符串，序列号生成规则结构

```

-----------------------64bit----------------------------
| appid (10bit) | time(30bit) | seq(15bit) | mac(9bit) |
--------------------------------------------------------

```

- appid: 应用程序分配的 ID
- time: time 是一个相对的时间，可以保证30年 
- seq: 累加值，保证每秒 3.2W 不重复
- mac: 机器ID

### 生成序列号

该序列号工具为非线程安全，在多线程环境中使用请使用锁来保证序列唯一或者每个线程分别一个 appid

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main() {
    adbase::Sequence seq;
    uint64_t seqId = seq.getSeqId(3, 3);
    std::string seqIdStr = adbase::base62Encode(seqId);
    LOG_INFO << seqId;
    LOG_INFO << seqIdStr;
    LOG_INFO << adbase::base62Decode(seqIdStr);
    return 0;
}
```
