## 简介

Supervisor（ http://supervisord.org ）是用Python开发的一个client/server服务，是Linux/Unix系 统下的一个进程管理工具，不支持Windows系统。

它可以很方便的监听、启动、停止、重启一个或多个进 程。

用Supervisor管理的进程，当一个进程意外被杀死，supervisort监听到进程死后，会自动将它重新拉 起，很方便的做到进程自动恢复的功能，不再需要自己写shell脚本来控制。 

## supervisor的组件

- supervisord(server 部分)：

  主要负责管理子进程，响应客户端命令以及日志的输出等； 

- supervisorctl(client 部分)：

  命令行客户端，用户可以通过它与不同的 supervisord 进程联系，获取子进程的状态等。 

- Web Server

  主要可以在界面上管理进程，Web Server其实是通过XML_RPC来实现的，可以向supervisor请求数据，也可以控制supervisor及子进程。配置在[inet_http_server]块里面; 

- XML_RPC接口

  提供了与webserver中相同的用于查询和控制进程的http接口; 