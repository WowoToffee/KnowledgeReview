# nacos数据模型

nacos 底层存储结构为:`Map<Namespace,<Map<group::serverName>, server>>`

![](https://cdn.nlark.com/yuque/0/2019/jpeg/338441/1561217857314-95ab332c-acfb-40b2-957a-aae26c2b5d71.jpeg)

## Namespace

Nacos引入了命名空间(`Namespace`)的概念来进行`多环境配置和服务`的管理及隔离。两个不同的命名空间不能相关调用，形成了隔离。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2xhcnNjaGVuZy9teUltZy9tYXN0ZXIvYmxvZ0ltZy9OYWNvcy8yMDE5MDcyMzE3NTkwOC5wbmc)

## GROUP

Nacos 中的一组配置集，是组织配置的维度之一。通过一个有意义的字符串（如 Buy 或 Trade ）对配置集进行分组，从而区分 Data ID 相同的配置集。当您在 Nacos 上创建一个配置时，如果未填写配置分组的名称，则配置分组的名称默认采用 DEFAULT_GROUP 。配置分组的常见场景：不同的应用或组件使用了相同的配置类型，如 database_url 配置和 MQ_topic 配置。