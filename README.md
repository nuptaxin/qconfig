# 目录

### QConfig

* [环境要求](doc/prerequest.md)
* [使用说明](doc/howto.md)
  - [使用api](doc/useWithApi.md)
  - [使用注解](doc/useWithAnnotation.md)
* [界面操作](doc/portal.md) 
  - [文件](doc/portal.md#文件)
  - [模版](doc/portal.md#模版)
  - [权限管理](doc/portal.md#权限管理)
* [API](doc/api.md)
* [最佳实践](doc/bestpractice.md)
* [模块说明](doc/code.md)
* [整体结构](doc/arch.md)
* [可用性](doc/ha.md)
* [部署](doc/start.md)
* [本地启动](doc/devStart.md)
* [开发](doc/dev.md)
* [监控](doc/monitor.md)
* [FAQ](doc/faq.md)

* 打包发布相关
  * 版本号升级
  > mvn versions:set -DgenerateBackupPoms=false -DnewVersion=0.5.0-SNAPSHOT-1.0.1
  * 打包
  > mvn clean package -Dmaven.test.skip=true -U
  * 部署(激活release标签)
  > mvn clean deploy -Dmaven.test.skip=true -P release