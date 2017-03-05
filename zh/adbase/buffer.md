# Buffer

Buffer 是基于 `std::vector<char>` 实现的二进制安全的 buffer 工具库，支持存取多种类型的数据, 支持自动分配内存空间，方便做数据流拼接, 内部结构如下：

```
  +-------------------+------------------+------------------+
  | prependable bytes |  readable bytes  |  writable bytes  |
  |                   |     (CONTENT)    |                  |
  +-------------------+------------------+------------------+
  |                   |                  |                  |
  0      <=      readerIndex   <=   writerIndex    <=     size
```

首先实例化对象时会初始化一块内存，默认读写标志都是0，分为 prependable 、readable、writable 三部分空间，prependable 是已读部分意味着可以覆盖再次写入部分，readable 是真正的数据部分，writable 是可以写入的空白空间, 该 buffer工具支持在前面 prepend 和 从后面 append 两种写入方式

##### 写入数据

prepend 操作前先使用 `prependableBytes` 方法回去当前可以 prepend 的长度，如果 prepend 的数据大于可以 prepend 的时候会终止程序

append 对长度没有限制，可以随意 append, buffer 工具会自动计算自动分配空间

prepend 和 append 操作支持写入数据类型为：`const char*`、`const void*`、`std::string`、`int32`、`int64`、`int16`、`int8` 

##### 读取数据

在从一个buffer 中读取数据的时候和写入数据类似，可以调用 `peek` 返回buffer 头部指针读取，读取完成后需要调用 `retrieve` 清空读取的数据，还有比较常用的是通过 `read*` 系列函数读取，该函数会自动清空数据，但是又的场景仅仅是拷贝一下数据，不想清空数据使用 `peek*` 系列函数，更多的函数请参考 开发手册


##### 举例

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main() {
    adbase::Buffer buffer;

    // write data
    LOG_INFO << "Init readable length: " << buffer.readableBytes();
    LOG_INFO << "Init writable length: " << buffer.writableBytes();

    const char* c = "bar";
    buffer.append(c, 3);
    LOG_INFO << "input char* readable length: " << buffer.readableBytes();
    LOG_INFO << "input char* writable length: " << buffer.writableBytes();

    std::string str = "hello";
    buffer.append(str);
    LOG_INFO << "input std::string readable length: " << buffer.readableBytes();
    LOG_INFO << "input std::string writable length: " << buffer.writableBytes();

    int64_t int64 = 123;
    buffer.appendInt64(int64);
    LOG_INFO << "input int64_t readable length: " << buffer.readableBytes();
    LOG_INFO << "input int64_t writable length: " << buffer.writableBytes();

    int32_t int32 = 123;
    buffer.appendInt32(int32);
    LOG_INFO << "input int32_t readable length: " << buffer.readableBytes();
    LOG_INFO << "input int32_t writable length: " << buffer.writableBytes();

    int16_t int16 = 123;
    buffer.appendInt16(int16);
    LOG_INFO << "input int16_t readable length: " << buffer.readableBytes();
    LOG_INFO << "input int16_t writable length: " << buffer.writableBytes();

    int8_t int8 = 123;
    buffer.appendInt8(int8);
    LOG_INFO << "input int8_t readable length: " << buffer.readableBytes();
    LOG_INFO << "input int8_t writable length: " << buffer.writableBytes();

    // read data
    LOG_INFO << std::string(buffer.peek(), 3);
    buffer.retrieve(3);
    LOG_INFO << "output char 3 readable length: " << buffer.readableBytes();
    LOG_INFO << "output char 3 writable length: " << buffer.writableBytes();

    LOG_INFO << std::string(buffer.peek(), 5);
    buffer.retrieve(5);
    LOG_INFO << "output char 5 readable length: " << buffer.readableBytes();
    LOG_INFO << "output char 5 writable length: " << buffer.writableBytes();

    LOG_INFO << buffer.readInt64();
    LOG_INFO << "output int64_t readable length: " << buffer.readableBytes();
    LOG_INFO << "output int64_t writable length: " << buffer.writableBytes();

    LOG_INFO << buffer.readInt32();
    LOG_INFO << "output int32_t readable length: " << buffer.readableBytes();
    LOG_INFO << "output int32_t writable length: " << buffer.writableBytes();

    LOG_INFO << buffer.readInt16();
    LOG_INFO << "output int16_t readable length: " << buffer.readableBytes();
    LOG_INFO << "output int16_t writable length: " << buffer.writableBytes();

    LOG_INFO << buffer.readInt8();
    LOG_INFO << "output int8_t readable length: " << buffer.readableBytes();
    LOG_INFO << "output int8_t writable length: " << buffer.writableBytes();
    return 0;
}
```

[GITHUB 代码](https://github.com/weiboad/adbase/blob/master/example/buffer.cpp)
