+++
title = "实例管理"
weight = 2
description = "查看微服务下的实例以及实例详情"
+++

## 实例管理

实例与微服务的关系：微服务与实例为一对多的从属关系。一个微服务可以有多个实例，一个实例只能属于一个微服务。

实例与配置的关系：配置与实例为一对多的应用关系。同一个微服务下，一个配置可以被多个实例应用，一个实例只能应用一个配置。

- **菜单层次**：全局层
- **菜单路径**：微服务管理 > 实例管理
- **默认角色**：平台管理员

## 实例详情

1. 点击查看详情→<img class="no-border" src="/docs/user-guide/system-configuration/microservice-management/image/info.png"/>按钮。
1. 点击‘实例信息’，查看实例的下列信息
    - 实例ID：实例ID的展示形式为`<主机名>:<所属微服务>:<端口号>`。实例ID具有唯一性，是实例的标识。
    - 主机名：分配给实例的地址。
    - IP：实例的实际地址。
    - 所属微服务：实例属于的微服务的微服务名称。
    - 端口号：实例的端口号。
    - 上次注册时间：实例上一层注册的时间。
    - 元数据：可以有多条数据项。每条数据项为名字-值的组合。
1. 点击‘配置环境信息’，查看实例的配置信息和环境信息。
    - 配置信息：实例所应用的配置的信息，以yml形式展示，不可修改。如需修改配置信息，请到配置管理中进行修改。
    - 环境信息：实例的环境信息，以yml形式展示，不可修改。