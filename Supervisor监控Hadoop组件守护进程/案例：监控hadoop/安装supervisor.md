## 切换到root用户

```shell
su root
```

## 添加EPEL存储库

```shell
yum install -y epel-release
```

## 安装pip

```shell
yum install -y python-pip
```

## 安装supervisor

```shell
pip install supervisor
```

## 创建supervisor配置文件目录并初始化配置文件

```shell
mkdir /etc/supervisor
echo_supervisord_conf > /etc/supervisor/supervisord.conf
```

## 修改supervisord.conf设置加载配置的路径

```shell
vi /etc/supervisor/supervisord.conf
```

修改以下内容

```
[include]
files = /etc/supervisor/*.conf
```

再修改以下内容（为后面使用supervisord-monitor进行网页端监控做准备，很重要）

```
[inet_http_server]
port=*:9001
username=user
password=123 
```

## 创建supervisord系统服务

```shell
vi /usr/lib/systemd/system/supervisord.service
```

修改为以下内容

```
[Unit]
Description=Process Monitoring and Control Daemon
After=rc-local.service

[Service]
Type=forking
Environment=JAVA_HOME=/opt/jdk1.8.0_144
ExecStart=/usr/bin/supervisord -c /etc/supervisor/supervisord.conf
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```

**注意事项：**

Environment这个配置项可以填写多个，用于加载需要的环境变量;

如果没有的话，要执行supervisord中监控管理的进程，会因为缺少环境变量而无法启动。

例如：此处要使用supervisord进程监控管理zookeeper进程，而zookeeper的启动依赖jdk，所以这里加入了JAVA_HOME的环境变量。

**Environment的填写，要根据实际情况填写**

## 使用命令控制supervisord系统服务

### 设置开机自启

```shell
systemctl enable supervisord
```

### 开启

```shell
systemctl start supervisord
```

### 重启

```shell
systemctl restart supervisord
```

### 停止

```she
systemctl stop supervisord
```

## 手动执行supervisor服务的命令

### 启动

表示使用配置文件/etc/supervisor/supervisord.conf来启动supervisord服务

```shell
supervisord -c /etc/supervisor/supervisord.conf
```

### 重新加载

表示supervisord服务重新加载*.conf中的配置

```shell
supervisorctl reload
```

### 停止所有supervisord监控的进程

```shell
supervisorctl stop all
```

### 启动所有supervisord可以监控的进程

```shell
supervisorctl start all
```

### 启动某个可以被监控的进程（例如zookeeper）
```shell
supervisorctl start zookeeper
```

