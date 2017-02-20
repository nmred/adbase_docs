# Adbase 开发手册

### Adbase

Adbase 是基于 C++11 开发的快速构建微服务高性能框架。由配置解析、基础工具库、网络库、Http Server、Memcache Server、Adhead Server、日志、度量工具等在开发微服务过程中常用的模块组成。Adbase 除了实现了自身定义的 Adhead RPC 协议外，还支持 Memcache、Http 等常用的 RPC 协议, 便于模块间的集成提升开发人员的开发效率。

### Adbase Seed

Adbase 在开发初期的一直遵循的原则就是如何使 C++ 开发人员简单、快速的后端服务。重点是兼顾服务质量与开发效率。所以 Adbase 在开发完成基础功能模块后开发了一个项目代码生成模块, 即 Adbase Seed。对于仅仅想使用 Adbase 基础组件的开发人员来说关于 Adbase Seed 部分可以跳过，不过建议阅读这部分，因为通过 Adbase Seed 生成的代码也可以为后面自定义使用 Adbase 提供参考。Seed 是通过日常项目实践将经常开发的微服务进行功能抽象，将常使用的模块如 RPC 服务、消息队列、Timer 定时器进行抽象提取，开发人员在开发一个新的项目时仅需要根据自己项目的需求进行配置上述模块即可生成项目骨架代码。开发人员重点关注业务需求即可，无需关心 RPC 等通用模块的实现以及项目构建部署等问题。提升开发人员的开发效率，并且对于降低入门 C++ 开发的门槛。

### 特性

- 快速开发、简单易用
- 高性能、多协议RPC 网络模块
- 完善的度量工具库
- 日志库支持
- 支持快速生成骨架代码
- 消息队列扩展支持
- Timer 定时器
- 兼容 Linux 32bit 64bit 平台
