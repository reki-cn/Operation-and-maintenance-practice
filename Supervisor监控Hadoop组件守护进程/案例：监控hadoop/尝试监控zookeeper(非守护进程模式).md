## 创建被监控程序的配置文件

首先在/etc/supervisor目录下创建一个“zookeeper.conf”文件

```shell
vi /etc/supervisor/zookeeper.conf
```

修改为以下内容

```
[program:zookeeper]     ;是被管理的进程配置参数，zookeeper是进程的名称,zookeeper是我要监控的对象
command=/home/hadoop/zookeeper-3.4.8/bin/zkServer.sh start-foreground   ; 程序启动命令(必须使用完整路径)
autostart=true       ; 在supervisord启动的时候也自动启动
startsecs=10         ; 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=unexpected     ; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外>杀死后才重启
startretries=3       ; 启动失败自动重试次数，默认是3
user=hadoop          ; 用哪个用户启动进程，默认是root
priority=999         ; 进程启动优先级，默认999，值小的优先启动
redirect_stderr=true ; 把stderr重定向到stdout，默认false
stdout_logfile_maxbytes=20MB  ; stdout 日志文件大小，默认50MB
stdout_logfile_backups = 20   ; stdout 日志文件备份数，默认是10
stdout_logfile=/etc/supervisor/zookeeper.log	;设置日志文件的路径
stopasgroup=true     ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=true     ;默认为false，向进程组发送kill信号，包括子进程
```

**注意事项：**

**最后两行配置：**

默认值是flase，改为true的意思是，如果supervisord进程被关闭之后，将会把它管理的所有子进程，也给关闭掉。

**配置项command：**

必须填写完整路径，因为如果是手动登录shell，运行supervisord进程，然后加载自定义的zookeeper程序配置，可以读取非完整路径的，但是**如果想要通过开机自启动的supervisord服务来启动zookeeper程序，必须设置完整路径**!

## 有关zookeeper的补充：

如果以start-foreground方式启动的zookeeper，是无法使用stop命令关闭的，

尝试zkServer.sh stop命令来停止以zkServer.sh start-foreground命令启动的zookeeper，会提示：

```Stopping zookeeper ... no zookeeper to stop (could not find file /home/hadoop/zookeeper-3.4.8/data/zookeeper_server.pid)```

说明stop命令是根据pid文件来关闭zookeeper进程的



前面监控zookeeper的方式，是以非守护进程的方式进行监控的，命令为
```zkServer.sh start-foreground```
所以不会生成相应的pid文件，当我们想要手动停止时，需要执行命令
```zkServer.sh stop```
但是此时会发现，因为找不到pid文件，而显示，没有zookeeper进程需要停止，但使用jps命令进行查看，会发现，仍然有zookeeper进程存在，具体进程名显示为```QuorumPeerMain```

另外还有一个zookeeper的启动使用的更加普遍，那就是```zkServer.sh start```，这是大家最常用的执行方式，但是**这个命令执行过后，zookeeper是以真正的守护进程的方式执行的**，supervisor如果用来监控这个命令，会出现一个情况，那就是第一次启动后，已经将zookeeper成功启动，但是用supervisorctl status 查看zookeeper进程时，会显示如下错误
```zookeeper  fatal  Exited too quickly (process log may have details)```
意思就是，虽然启动了，但是很快就退出了，如果此时要通过supervisord进程来关闭刚启动的zookeeper进程，会发现无法做到。

原因是supervisord只能用来监控管理普通进程和普通的后台非守护进程

解决办法就是借助自行编写一个PID文件的前台监控程序与后台进程相关联，然后由supervisord进程来监控这个前台监控程序