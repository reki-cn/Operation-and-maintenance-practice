## supervisor功能

参考文档“supervisor安装配置.pdf”可知，使用supervisor可以对单台机器上的进程进行监控，并且可以有可视化的界面，另外可以搭配supervisor monitor，对由多台机器构成的整个集群进行监控。

## supervisor的问题
supervisor监控的都是非守护后台进程和普通进程，对于后台守护进程就无能为力了。

