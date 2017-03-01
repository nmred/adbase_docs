#Enbain

端模式（Endian）的这个词出自Jonathan Swift书写的《格列佛游记》。这本书根据将鸡蛋敲开的方法不同将所有的人分为两类，从圆头开始将鸡蛋敲开的人被归为Big Endian，从尖头开始将鸡蛋敲开的人被归为Littile Endian。小人国的内战就源于吃鸡蛋时是究竟从大头（Big-Endian）敲开还是从小头（Little-Endian）敲开。在计算机业Big Endian和Little Endian也几乎引起一场战争。在计算机业界，Endian表示数据在存储器中的存放顺序。采用大端方式 进行数据存放符合人类的正常思维，而采用小端方式进行数据存放利于计算机处理。下文举例说明在计算机中大小端模式的区别

	小端口诀: 高高低低 -> 高字节在高地址, 低字节在低地址
	大端口诀: 高低低高 -> 高字节在低地址, 低字节在高地址

```
long test = 0x313233334;
小端机器:
低地址 -->　高地址
00000010: 34 33 32 31         -> 4321

大端机器:
低地址 -->　高地址
00000010: 31 32 33 34         -> 4321
```

一般在网络通信中使用大端模式，所以 Adbase Enbain 工具是封装了机器码转化网络编码模式的工具方法,如下是转化工具的使用方法

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

std::string debug64(uint64_t value) {
    uint64_t base = 0x01;
    std::string str = "";
    for (int i = 63; i >= 0; i--) {
        str += (value & (base << i)) ? "1 " : "0 ";
    }

    return str;
}
std::string debug32(uint64_t value) {
    uint64_t base = 0x01;
    std::string str = "";
    for (int i = 31; i >= 0; i--) {
        str += (value & (base << i)) ? "1 " : "0 ";
    }

    return str;
}
std::string debug16(uint64_t value) {
    uint64_t base = 0x01;
    std::string str = "";
    for (int i = 15; i >= 0; i--) {
        str += (value & (base << i)) ? "1 " : "0 ";
    }

    return str;
}

int main() {
    uint64_t value = 12345;
    LOG_INFO << "before convert network enbian:" << debug64(value);
    uint64_t valuen = adbase::hostToNetwork64(value);
    LOG_INFO << "after convert network enbian:" << debug64(valuen);
    uint64_t valueh = adbase::networkToHost64(valuen);
    LOG_INFO << "reconvert network enbian:" << debug64(valueh);

    uint32_t value1 = 12345;
    LOG_INFO << "before convert network enbian:" << debug32(value1);
    uint32_t value1n = adbase::hostToNetwork32(value1);
    LOG_INFO << "after convert network enbian:" << debug32(value1n);
    uint32_t value1h = adbase::networkToHost32(value1n);
    LOG_INFO << "reconvert network enbian:" << debug32(value1h);

    uint16_t value2 = 12345;
    LOG_INFO << "before convert network enbian:" << debug16(value2);
    uint16_t value2n = adbase::hostToNetwork16(value2);
    LOG_INFO << "after convert network enbian:" << debug16(value2n);
    uint16_t value2h = adbase::networkToHost16(value2n);
    LOG_INFO << "reconvert network enbian:" << debug16(value2h);
    return 0;
}
```
