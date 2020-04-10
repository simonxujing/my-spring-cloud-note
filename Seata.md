# 一、简介

​	Seata是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。保证全局的数据一致性。

分布式事务处理过程的1ID+3组件模型：

- Transaction ID：全局唯一事务ID

- TC - 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或回滚。

- TM - 事务管理器：定义全局事务的范围：开始全局事务、提交或回滚全局事务。

- RM - 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

# 二、seata的安装

## 1、下载解压后，备份file.conf文件，修改file.conf文件

​	主要修改自定义事务组名称、事务日志存储模式为db、数据库连接信息

### 1）修改事务组名

```conf
service {
  #vgroup->rgroup
  vgroup_mapping.my_test_tx_group = "fsp_tx_group"
  ...
}
```

### 2）store修改

```conf
store {
  ## store mode: file、db
  mode = "db"
  ...
```

### 3）数据库连接信息

```conf
## database store
  db {
    ...
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "root"
    password = "xu691004."
    ...
  }
```

## 2、建数据库seata、建表

​	建表文件目录地址：seata\conf\db_store.sql

## 3、registry.conf修改

修改为使用nacos

```conf
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    serverAddr = "10.162.67.13:1111"
    namespace = ""
    cluster = "default"
  }
  ...
}
```

