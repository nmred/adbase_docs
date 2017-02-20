# 开发项目

### 项目配置

由于业务需求是支持多个关键字字典的匹配服务，所以我们要允许项目启动的时候通过配置文件配置关键词字典名称和对应的字典路劲，这样面临的问题是如何在 Adbase Seed 骨架代码上增加配置解析和配置读取以及配置定义

##### 配置定义

在骨架代码中增加一个全局的配置项，需要在 ini 配置文件增加配置项和在 AdbaseConfig 这个结构体总增加定义

- 修改 conf/system.ini 文件增加匹配服务本身的配置项，用来配置匹配服务支持的字典

```
[pattern]
dicta=./dicta.txt
dictb=./dictb.txt
```
[GITHUB代码段]()

- 修改 src/AdbaseConfig.hpp 在 `AdbaseConfig` 这个结构体中增加属性 patternConfig 用来存储匹配关键字字典的映射关系

```c
std::unordered_map<std::string, std::string> patternConfig;
```
[GITHUB代码段]()

-  配置解析

在定义好配置项后下一步是通过解析 ini 配置文件，将配置文件中的配置项和 `AdbaseConfig` 这个全局的配置变量中的配置项关联起来，这块在骨架代码中启动的时候会将 Ini 配置文件解析，具体的 `AdbaseConfig` 关联是通过回调 `App->loadConfig` 这个方法实现的，所以我们要解析一个配置到全局配置中，需要在这个方法中实现，具体的操作如下：

修改后的 `App->loadCondig` 代码如下

```c
void App::loadConfig(adbase::IniConfig& config) {
    // 解析词库配置
    std::vector<std::string> patternKeys  = config.Options("pattern");
    std::unordered_map<std::string, std::string> patternConfig;
    for (auto & t : patternKeys) {
        patternConfig[t] = config.GetOption("pattern", t);
    }
    _configure->patternConfig = patternConfig;
}
```
[GITHUB代码段]()

- 配置读取

在解析了

### 匹配服务核心逻辑实现

### 开发 HTTP 接口

### 实现 Memcache get

