# 三分钟内在本地环境部署、启动QConfig配置中心。

考虑到Docker的便捷性，我们还提供了Quick Start的Docker版本，如果你对Docker比较熟悉的话，可以参考[QConfig Quick Start Docker部署](quick-start-docker.md)通过Docker快速部署QConfig。

不过这里需要注意的是，Quick Start只针对本地测试使用，如果要部署到生产环境，还请另行参考[分布式部署指南](distributed-deployment-guide.md)。

> 注：Quick Start需要有bash环境，Windows用户请安装[Git Bash](https://git-for-windows.github.io/)，建议使用最新版本，老版本可能会遇到未知问题。也可以直接通过IDE环境启动，详见[QConfig开发指南](../devStart.md)。

# &nbsp;
# 一、准备工作
## 1.1 Java

* QConfig服务端：1.7+
* QConfig客户端：1.7+

由于Quick Start会在本地同时启动服务端和客户端，所以需要在本地安装Java 1.7+。

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

Quick Start只针对本地测试使用，用户需要使用源代码打包，参考如下步骤：

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

## 2.2 配置数据库连接信息
1. 删除原有的ROOT文件夹和admin文件夹
```shell
rm -rf /usr/local/apache-tomcat-8.5.60/webapps/ROOT/
rm -rf /usr/local/apache-tomcat-8.5.60/webapps/admin/
```

2. 复制步骤1.3中打包生成的server/target下的ROOT.war到本地安装的Tomcat的webapp目录下
```shell
cp /Users/renzhengxin/IdeaProjects/nuptaxin/qconfig/server/target/ROOT.war /usr/local/apache-tomcat-8.5.60/webapps/ROOT.war
cp /Users/renzhengxin/IdeaProjects/nuptaxin/qconfig/admin/target/ROOT.war /usr/local/apache-tomcat-8.5.60/webapps/admin.war
```

> 注意：填入的用户需要具备对qconfig数据的读写权限。

# 三、启动QConfig配置中心
## 3.1 确保端口未被占用
Quick Start会在本地启动2个服务，使用8080端口，请确保端口当前没有被使用。

例如，在Linux/Mac下，可以通过如下命令检查：
```sh
lsof -i:8080
```

## 3.1.1 安装Jetty
> https://www.eclipse.org/jetty/documentation/jetty-9/index.html#runner

## 3.2 Jetty启动Server
```sh
java -jar /Users/renzhengxin/.m2/repository/org/eclipse/jetty/jetty-runner/9.4.9.v20180320/jetty-runner-9.4.9.v20180320.jar --port 8080 server/target/ROOT.war
```
访问http://localhost:8080，如果出现erueka界面，即表示已正常启动

### 3.2.1 Jetty启动Admin
```sh
java -jar /Users/renzhengxin/.m2/repository/org/eclipse/jetty/jetty-runner/9.4.9.v20180320/jetty-runner-9.4.9.v20180320.jar --port 8081 admin/target/ROOT.war
```
访问http://localhost:8081/webapp/page/index.html，如果出现登录界面，即表示已正常启动
> 启动后，默认登陆账户为 admin 密码为 123456

## 3.3 注意
Quick Start只是用来帮助大家快速体验QConfig项目，具体实际使用时请参考：[分布式部署指南](distributed-deployment-guide.md)。

另外需要注意的是Quick Start不支持增加环境，只有通过分布式部署才可以新增环境，同样请参考：[分布式部署指南](distributed-deployment-guide.md)

# 四、使用QConfig配置中心
## 4.1 使用样例项目

### 4.1.1 查看样例配置
1. 打开http://localhost:8081/webapp/page/index.html#/qconfig

2. 输入用户名admin，密码123456后登录

3. 点击qconfig进入配置界面，可以看到当前有一个多个配置文件

### 4.1.2 运行客户端程序
我们准备了一个简单的[Demo客户端](../../demo), appCode=b_qconfig_test来演示从QConfig配置中心获取配置。

程序很简单，就是从本地或者配置中心读取各种配置文件。

获取token：访问 http://localhost:8081/webapp/page/index.html#/qconfig/appinfo 输入应用appCode获取token，并配置到app-info.properties文件中
添加配置文件：
http://localhost:8081/webapp/page/index.html#/qconfig/b_qconfig_test/dev:?groupName=b_qconfig_test
添加配置文件, 并存储适当的配置内容（如果想配置其它文件，先删除qconfig_test目录下对应的配置文件，然后在配置中心添加）： dcdc.properties

运行`./demo.sh client`启动Demo客户端：
```sh
java -jar /Users/renzhengxin/.m2/repository/org/eclipse/jetty/jetty-runner/9.4.9.v20180320/jetty-runner-9.4.9.v20180320.jar --port 8082 demo/target/ROOT.war
```
看到控制台输出use remote file, name=dcdc.properties，以及配置文件中的所有内容时，证明成功。

### 4.1.3 修改配置并发布

1. http://localhost:8081/webapp/page/index.html#/qconfig/b_qconfig_test/dev
