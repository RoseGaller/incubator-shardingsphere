+++
pre = "<b>3.3.2. </b>"
toc = true
title = "注册中心"
weight = 2
+++

## 实现动机

- 相对于配置中心管理配置数据，注册中心存放运行时的动态/临时状态数据，比如可用的proxy的实例，需要禁用或熔断的datasource实例。

- 通过注册中心，可以提供熔断数据库访问程序对数据库的访问和禁用从库的访问的编排治理能力。治理仍然有大量未完成的功能（比如流控等）。

## 注册中心数据结构

注册中心在定义的命名空间的state下，创建数据库访问对象运行节点，用于区分不同数据库访问实例。包括instances和datasources节点。

```
instances
    ├──your_instance_ip_a@-@your_instance_pid_x
    ├──your_instance_ip_b@-@your_instance_pid_y
    ├──....
datasources
    ├──ds0
    ├──ds1
    ├──....
```
Sharding-Proxy支持多逻辑数据源，因此datasources子节点的名称采用schema_name.data_source_name的形式。
```
instances
    ├──your_instance_ip_a@-@your_instance_pid_x
    ├──your_instance_ip_b@-@your_instance_pid_y
    ├──....
datasources
    ├──sharding_db.ds0
    ├──sharding_db.ds1
    ├──....
```
### state/instances

数据库访问对象运行实例信息，子节点是当前运行实例的标识。
运行实例标识由运行服务器的IP地址和PID构成。运行实例标识均为临时节点，当实例上线时注册，下线时自动清理。
注册中心监控这些节点的变化来治理运行中实例对数据库的访问等。

### state/datasources

可以治理读写分离从库，可动态添加删除以及禁用。

## 操作指南

### 熔断实例

可在IP地址@-@PID节点写入`DISABLED`（忽略大小写）表示禁用该实例，删除DISABLED表示启用。

Zookeeper命令如下：

```
[zk: localhost:2181(CONNECTED) 0] set /your_zk_namespace/your_app_name/state/instances/your_instance_ip_a@-@your_instance_pid_x DISABLED
```

### 禁用从库

在读写分离（或数据分片+读写分离）场景下，可在数据源名称子节点中写入`DISABLED`（忽略大小写）表示禁用从库数据源，删除DISABLED或节点表示启用。

Zookeeper命令如下：

```
[zk: localhost:2181(CONNECTED) 0] set /your_zk_namespace/your_app_name/state/datasources/your_slave_datasource_name DISABLED
```
