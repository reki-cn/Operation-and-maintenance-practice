为了解决这个问题，我们需要

一些在前台运行的程序，它会在守护进程退出时退出，并且还会向守护进程代理信号。 

考虑使用以下bash脚本：

```shell
#!/bin/bash
set -eu

pidfile="/var/run/your-daemon.pid"

command="/usr/sbin/your-daemon"

function kill_app(){
 kill $(cat $pidfile)
 exit 0
}

trap "kill_app" SIGINT SIGTERM

$command

sleep 2

while [ -f $pidfile ] && kill -0 $(cat $pidfile) ; do
 sleep 0.5
done

exit 1000
```

脚本解释：

```shell
#!/bin/bash
# set -e ：表示这行代码之后的任何代码，如果返回一个非0的值，那么整个脚本立即退出，官方的说明是为了防止错误出现滚雪球的现象。
# 参考：https://segmentfault.com/q/1010000000302700
# set -u : 表示脚本在头部加上它，遇到不存在的变量就会报错，并停止执行。
# 参考：https://www.xttblog.com/?p=2339
set -eu

# 定义一个pidfile变量
# pidfile变量存储的值是要监控的守护进程在运行时产生的pid文件的绝对路径，用双引号括起来
# 补充：pidfile中的内容是守护进程的进程ID
pidfile="/var/run/your-daemon.pid"

# command变量存储的值是要用来启动守护进程的命令，这个路径也必须是绝对路径，用双引号括起来
command="/usr/sbin/your-daemon"

# 定义一个代理信号的函数，kill_app
function kill_app(){
# 从pidfile中读取守护进程的进程ID，并用kill命令将其杀死，从而达到杀掉守护进程的目的
 kill $(cat $pidfile)
# exit 0 表示退出当前整个shell脚本
# 参考：https://blog.csdn.net/feixiaohuijava/article/details/53419721
 exit 0
}

# trap 是用来捕捉信号的命令，这里是捕捉kill_app函数返回的信号
# SIGINT：程序中断(interrupt)信号
# SIGTERM：程序结束(terminate)信号
# 这行语句意思是，当捕捉到kill_app函数的信号时，立刻做出SIGINT和SIGTERM的行为
# 参考：https://www.cnblogs.com/f-ck-need-u/p/7454174.html
trap "kill_app" SIGINT SIGTERM

# 运行守护进程
$command

# 让当前脚本休眠2秒，注意单位是秒
# 参考：https://www.cnblogs.com/senior-engineer/p/6203292.html
sleep 2

# 下面的循环作用：每0.5秒检查一次：pid文件是否正常存在，并且守护进程的进程ID是否存在

# -f $pidfile:检查$pidfile是否为常规文件
# 参考：https://www.cnblogs.com/senior-engineer/p/6206329.html
# kill -0 $(cat $pidfile)：根据守护进程文件中的进程ID来检查守护进程是否存在
# 参考：https://blog.51cto.com/dzm911/1940295
while [ -f $pidfile ] && kill -0 $(cat $pidfile) ; do
# 如果满足以上条件，每0.5秒循环一次
 sleep 0.5
done

# 表示异常退出
exit 1000
```

