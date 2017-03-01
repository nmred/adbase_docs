# 系统相关

### 文件系统

##### 目录相关

adbase 提供递归创建目录 `adbase::mkdirRecursive` 和递归遍历目录 `adbase::recursiveDir`

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main(void) {

    // 目录路劲
    // 目录权限
    // 目录递归
    if (!adbase::mkdirRecursive("./bar/foo/aa/bb", 0755, true)) {
        LOG_INFO << "mkdir fail.";
        return 0;
    }

    std::vector<std::string> excludes;
    std::vector<std::string> dirs;
    adbase::recursiveDir("./", true, excludes, dirs);
    for (auto &t : dirs) {
        LOG_INFO << t;
    }
    return 0;
}
```

##### 文件读取

文件读取通过 `ReadSmallFile` 工具类获取文件中的数据, 并且根据 `ReadSmallFile` 封装了方法 `readFile`

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main(void) {
    std::string filename = "/proc/self/status";
    std::string status;
    adbase::readFile(filename, 65536, &status);
    LOG_INFO << status;

    std::string status1;
    adbase::ReadSmallFile readfile(filename);
    readfile.readToString(65536, &status1, nullptr, nullptr, nullptr);
    LOG_INFO << status;

    adbase::ReadSmallFile readfileBuffer(filename);
    int size = 0;
    readfileBuffer.readToBuffer(&size);
    const char* buffer = readfileBuffer.buffer();
    LOG_INFO << "read file size:" << size;
    LOG_INFO << "read file content:" << buffer;

    return 0;
}
```

##### 文件写入

文件写入通过`AppendFile` 工具类实现，通过 `AppendFile->append` 写入数据， `AppendFile->flush` 刷新数据， `AppendFile->writtenBytes` 获取写入的数据长度

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main(void) {
    std::string filename = "./bar";
    adbase::AppendFile fp(filename);
    std::string data = "test";
    fp.append(data.c_str(), data.size());
    fp.flush();
    LOG_INFO << "write file size:" << fp.writtenBytes();

    return 0;
}
```

### 进程相关

##### 获取进程信息

获取进程 ID 已经进程名称

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main(void) {
    // get pid
    LOG_INFO << "process id:" << adbase::pid();
    LOG_INFO << "process id:" << adbase::pidString();
    LOG_INFO << "process name:" << adbase::procname();

    return 0;
}
```

##### 获取网络相关

获取主机名以及网卡的IP及Mac地址

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main(void) {
    LOG_INFO << "host name:" << adbase::hostname();
    std::unordered_map<std::string, std::unordered_map<std::string, std::string>> ifconfigs = adbase::ifconfig();
    for (auto &t : ifconfigs) {
        LOG_INFO << "Network name:" << t.first;
        for (auto &info : t.second) {
            LOG_INFO << "info->" << info.first << ":" << info.second;
        }
    }
    return 0;
}
```

其中 `adbase::ifconfig` 会返回多张网卡的 ip 和 mac 地址

##### 获取进程运行信息

获取进程运行的信息，包含线程数、内存、打开文件数等信息，进程运行信息是根据系统文件`/proc/self/fd`、`/proc/self/status` 和 `/proc/self/stat` 获取计算得出, status 具体含义请参考 `proc/self/status` 字段含义

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main(void) {
    LOG_INFO << "proc open file:" << adbase::procFdNum();
    LOG_INFO << "proc status:";

    std::unordered_map<std::string, std::string> status = adbase::procStats();
    for (auto &t : status) {
        LOG_INFO << "   " << t.first << ":" << t.second;
    }
    return 0;
}
```
