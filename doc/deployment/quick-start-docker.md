# 一、准备工作
## 1.1 Java

* QConfig服务端：1.7+
* QConfig客户端：1.7+

由于Quick Start Docker需要在本地打包生成war包，所以需要在本地安装Java 1.7+。

在配置好后，可以通过如下命令检查：
```sh
java -version
```

样例输出：
```sh
java version "1.7.0_74"
Java(TM) SE Runtime Environment (build 1.7.0_74-b02)
Java HotSpot(TM) 64-Bit Server VM (build 25.74-b02, mixed mode)
```

Windows用户请确保JAVA_HOME环境变量已经设置。

## 1.2 MySQL

* 版本要求：5.6.5+

QConfig的表结构对`timestamp`使用了多个default声明，所以需要5.6.5以上版本。

连接上MySQL后，可以通过如下命令检查：
```sql
SHOW VARIABLES WHERE Variable_name = 'version';
```

| Variable_name | Value  |
|---------------|--------|
| version       | 5.7.11 |

### 1.3 手动打包Quick Start安装包

Quick Start Docker需要用户修改部分配置，用户需要使用源代码打包，参考如下步骤：

* 在根目录下执行`mvn clean package -Dmaven.test.skip=true -U`

# 二、安装步骤
## 2.1 创建数据库
QConfig服务端需要数据库：`qconfig`，我们把数据库、表的创建和样例数据准备了sql文件，只需要导入数据库即可。

> 注意：如果你本地已经创建过QConfig数据库，请注意备份数据。我们准备的sql文件会清空QConfig相关的表。

### 2.1.1 创建qconfig
通过各种MySQL客户端导入[scripts/docker/sql/main.sql](./../../scripts/docker/sql/main.sql)即可。

下面以MySQL原生客户端为例：
```sql
source /your_local_path/sql/main.sql
```

导入成功后，可以通过执行以下sql语句来验证：
```sql
select * from qconfig.pb_app;
```

| id | code           | name           |
|----|----------------|----------------|
| 1  | b_qconfig_test | b_qconfig_test |

> 注意：填入的用户需要具备对qconfig数据的读写权限。

# 三、启动QConfig配置中心
## 3.1 修改qconfig-server数据库连接配置

> server/src/main/resources/qconfig_test/qconfig/mysql.properties

修改jdbc.url/jdbc.username/jdbc.username/jdbc.password的配置

## 3.2 重新打包
```sh
mvn clean package -Dmaven.test.skip=true -U
```

### 3.3 重新生成docker镜像
将/doc/deployment/qconfig-server/Dockerfile和server/target/ROOT.war拷贝到同一目录下
重新打包
```sh
docker build -t okracode/qconfig-quick-start:server .
```

## 3.4 新建replicaSet并运行
```shell
kubectl apply -f qconfig-server-rs.yaml
```
查看日志
```shell
kubectl logs qconfig-server-rs-4px7g
```
查看是否运行成功：
```shell
kubectl port-forward rs/qconfig-server-rs 8080:8080 --address 0.0.0.0
```
访问IP：8080端口查看页面是否正常启动

查看日志：
```shell
kubectl get po
kubectl exec -it qconfig-server-rs-pwf7h sh
tail -f logs/qconfigserver.2021-09-30-14.log
```

## 3.5 新建服务用于内部访问
```shell
kubectl apply -f qconfig-server-svc.yaml
```

# 四、启动QConfig配置中心管理后台
## 4.1 修改qconfig-admin数据库连接配置

> admin/src/main/resources/qconfig_test/qconfig/mysql.properties

修改jdbc.url/jdbc.username/jdbc.username/jdbc.password的配置

## 4.2 重新打包
```sh
mvn clean package -Dmaven.test.skip=true -U
```

### 4.3 重新生成docker镜像
将/doc/deployment/qconfig-admin/Dockerfile和admin/target/ROOT.war拷贝到同一目录下
重新打包
```sh
docker build -t okracode/qconfig-quick-start:admin .
```

## 4.4 新建replicaSet并运行
```shell
kubectl apply -f qconfig-admin-rs.yaml
```
查看日志
```shell
kubectl logs qconfig-admin-rs-4px7g
```
查看是否运行成功：
```shell
kubectl port-forward rs/qconfig-admin-rs 8080:8083 --address 0.0.0.0
```
访问IP：8080端口查看页面是否正常启动

输入用户名admin，密码123456后登录

查看日志：
```shell
kubectl get po
kubectl exec -it qconfig-server-rs-pwf7h sh
tail -f logs/qconfigserver.2021-09-30-14.log
```

## 4.5 新建服务用于内部访问
```shell
kubectl apply -f qconfig-admin-svc.yaml
```

## 4.6 修改Ingress将qconfig.okracode.com请求转发到qconfig-admin
```shell
- host: qconfig.okracode.com
  http:
    paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: qconfig-admin
            port:
              number: 80
```
访问http://qconfig.okracode.com/即可到qconfig配置中心对应后台

## 五、运行客户端程序
我们准备了一个简单的[Demo客户端](../../demo), appCode=b_qconfig_test来演示从QConfig配置中心获取配置。

程序很简单，就是从本地或者配置中心读取各种配置文件。

获取token：访问 http://localhost:8081/webapp/page/index.html#/qconfig/appinfo 输入应用appCode获取token，并配置到app-info.properties文件中
添加配置文件：
http://localhost:8081/webapp/page/index.html#/qconfig/b_qconfig_test/dev:?groupName=b_qconfig_test
添加配置文件, 并存储适当的配置内容（如果想配置其它文件，先删除qconfig_test目录下对应的配置文件，然后在配置中心添加）： dcdc.properties