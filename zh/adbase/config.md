# 解析配置

### 解析

解析配置功能模块目前框架支持 INI 格式的配置文件或者字符串解析, 通过 ` adbase::IniParse` 来完成整体的解析，返回 `adbase::IniConfig` 对象，通过 `adbase::IniConfig` 对象中的方法灵活的获取配置数据，例如：

```cpp
#include <adbase/Config.hpp>
#include <adbase/Logging.hpp>

int main() {
    std::string ini = "[http]\nhost=127.0.0.1\nport=80\ntimeout=3\ndaemon=yes\n";
    adbase::IniConfig config = adbase::IniParse::load(ini);
    LOG_INFO << config.getOption("http", "host");
    LOG_INFO << config.getOptionUint32("http", "port");
    LOG_INFO << config.getOptionUint32("http", "timeout");
    LOG_INFO << config.getOptionBool("http", "daemon");
    return 0;
}
```

[GITHUB 代码](https://github.com/weiboad/adbase/blob/master/example/config.cpp)

配置模块也支持解析文件例如：

```cpp
adbase::IniConfig config = adbase::IniParse::loadFile(iniFile);
```

### 获取配置数据

通过 `adbase::IniParse` 将配置解析后返回 `adbase::IniConfig` 对象, 下面介绍一个 IniConfig 对象的常用的方法

##### 获取单个配置

```cpp
adbase::IniConfig config = adbase::IniParse::loadFile(iniFile);
std::string host = config.getOption("http", "host");
```

默认使用 `getOption` 方法返回时字符串类型，如果获取 int 类型的使用 `getOptionUint32` 或者 `getOptionUint64`, 获取 bool 类型的使用 `getOptionBool`

##### 获取整节配置

有的时候我们对具体的配置项名称是未知的仅仅知道配置节的名称，如下是 ini 示例，假如仅仅知道节名 `adbase` 需要获其下的配置数据，那么就需要获取到该节下的所有配置项 Key, 使用`options` 可以获取到

```
; ini config
[adbase]
bar=123
foo=455
```

```cpp
adbase::IniConfig config = adbase::IniParse::loadFile(iniFile);
std::vector< std::string > options = config.options("adbase");

for(auto & key : options) {
	LOG_INFO << config.getOptionUint64("adbase", key);
}
```

##### 获取所有节

当对配置节都是未知的时候，可以使用 `sections` 方法将所有配置节获取出来

```cpp
adbase::IniConfig config = adbase::IniParse::loadFile(iniFile);
std::vector< std::string > sections = config.sections();
```
