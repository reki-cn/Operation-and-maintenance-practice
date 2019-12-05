## 添加配置文件

```shell
vi /etc/supervisor/zookeeper.conf
```

```
[program:zookeeper]
command=/bin/bash /home/hadoop/monitor/zookeeper_monitor.sh   ; 程序启动命令(必须使用完整路径)
autostart=false       ; 在supervisord启动的时候也自动启动
startsecs=10         ; 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=unexpected     ; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外>杀死后才重启
startretries=3       ; 启动失败自动重试次数，默认是3
user=root          ; 用哪个用户启动进程，默认是root
priority=999         ; 进程启动优先级，默认999，值小的优先启动
redirect_stderr=false    ; 把stderr重定向到stdout，默认false
stdout_logfile_maxbytes=20MB  ; stdout 日志文件大小，默认50MB
stdout_logfile_backups = 20   ; stdout 日志文件备份数，默认是10
stdout_logfile=/etc/supervisor/zookeeper.log    ;设置日志文件的路径
stdout_errfile_maxbytes=20MB  ; stdout 日志文件大小，默认50MB
stdout_errfile_backups = 20   ; stdout 日志文件备份数，默认是10
stdout_errfile=/etc/supervisor/zookeeper.err.log    ;设置日志文件的路径
stopasgroup=false     ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false     ;默认为false，向进程组发送kill信号，包括子进程
```

**注意：**
**redirect_stderr=false;**这个配置项不要设置成**redirect_stderr=true;**

否则后续在用supervisor-monitor进行监控时，日志会出现无法读取的情况；

所以其它的各个应用进程的supervisor的此处配置也要用false，方便使用supervisor-monitor进行监控

## 添加守护进程监控脚本

```shell
vi /home/hadoop/monitor/zookeeper_monitor.sh
```

```
#!/bin/bash
set -eu
pidfile="/home/hadoop/zookeeper-3.4.8/data/zookeeper_server.pid"
command="/home/hadoop/zookeeper-3.4.8/bin/zkServer.sh start"
# Proxy signals
function kill_app(){
 kill $(cat $pidfile)
 exit 0 # exit okay
}
trap "kill_app" SIGINT SIGTERM
# Launch daemon
su hadoop -s $command
sleep 5
# Loop while the pidfile and the process exist
while [ -f $pidfile ] && kill -0 $(cat $pidfile) ; do
 sleep 0.5
done
exit 1000 # exit unexpected
```

