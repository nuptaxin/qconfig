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
1. 打开http://localhost:8070

> Quick Start集成了[Spring Security简单认证](zh/development/portal-how-to-implement-user-login-function#实现方式一：使用QConfig提供的spring-security简单认证)，更多信息可以参考[Portal 实现用户登录功能](zh/development/portal-how-to-implement-user-login-function)

<img src="https://github.com/nobodyiam/QConfig-build-scripts/raw/master/images/QConfig-login.png" alt="登录" width="640px">

2. 输入用户名QConfig，密码admin后登录

![首页](https://raw.githubusercontent.com/nobodyiam/QConfig-build-scripts/master/images/QConfig-sample-home.png)

3. 点击SampleApp进入配置界面，可以看到当前有一个配置timeout=100
   ![配置界面](https://raw.githubusercontent.com/nobodyiam/QConfig-build-scripts/master/images/sample-app-config.png)

> 如果提示`系统出错，请重试或联系系统负责人`，请稍后几秒钟重试一下，因为通过Eureka注册的服务有一个刷新的延时。

### 4.1.2 运行客户端程序
我们准备了一个简单的[Demo客户端](https://github.com/ctripcorp/QConfig/blob/master/QConfig-demo/src/main/java/com/ctrip/framework/QConfig/demo/api/SimpleQConfigConfigDemo.java)来演示从QConfig配置中心获取配置。

程序很简单，就是用户输入一个key的名字，程序会输出这个key对应的值。

如果没找到这个key，则输出undefined。

同时，客户端还会监听配置变化事件，一旦有变化就会输出变化的配置信息。

运行`./demo.sh client`启动Demo客户端，忽略前面的调试信息，可以看到如下提示：
```sh
QConfig Config Demo. Please input key to get the value. Input quit to exit.
>
```
输入`timeout`，会看到如下信息：
```sh
> timeout
> [SimpleQConfigConfigDemo] Loading key : timeout with value: 100
```

> 如果运行客户端遇到问题，可以通过修改`client/log4j2.xml`中的level为DEBUG来查看更详细日志信息
> ```xml
> <logger name="com.ctrip.framework.QConfig" additivity="false" level="trace">
>     <AppenderRef ref="Async" level="DEBUG"/>
> </logger>
> ```

### 4.1.3 修改配置并发布

1. 在配置界面点击timeout这一项的编辑按钮
   ![编辑配置](https://raw.githubusercontent.com/nobodyiam/QConfig-build-scripts/master/images/sample-app-modify-config.png)

2. 在弹出框中把值改成200并提交
   ![配置修改](https://raw.githubusercontent.com/nobodyiam/QConfig-build-scripts/master/images/sample-app-submit-config.png)

3. 点击发布按钮，并填写发布信息
   ![发布](https://raw.githubusercontent.com/nobodyiam/QConfig-build-scripts/master/images/sample-app-release-config.png)

![发布信息](https://raw.githubusercontent.com/nobodyiam/QConfig-build-scripts/master/images/sample-app-release-detail.png)

### 4.1.4 客户端查看修改后的值
如果客户端一直在运行的话，在配置发布后就会监听到配置变化，并输出修改的配置信息：
```sh
[SimpleQConfigConfigDemo] Changes for namespace application
[SimpleQConfigConfigDemo] Change - key: timeout, oldValue: 100, newValue: 200, changeType: MODIFIED
```

再次输入`timeout`查看对应的值，会看到如下信息：
```sh
> timeout
> [SimpleQConfigConfigDemo] Loading key : timeout with value: 200
```

## 4.2 使用新的项目
### 4.2.1 应用接入QConfig
这部分可以参考[Java应用接入指南](../usage/java-sdk-user-guide)

### 4.2.2 运行客户端程序
由于使用了新的项目，所以客户端需要修改appId信息。

编辑`client/META-INF/app.properties`，修改app.id为你新创建的app id。
```properties
app.id=你的appId
```
运行`./demo.sh client`启动Demo客户端即可。
