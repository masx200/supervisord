[![Go Report Card](https://goreportcard.com/badge/github.com/ochinchina/supervisord)](https://goreportcard.com/report/github.com/ochinchina/supervisord)

# 为什么创建这个项目？

Python 脚本 supervisord 是一个强大的工具，被很多人用来管理进程。我也喜欢
supervisord。

但是这个工具需要在目标系统中安装庞大的 Python 环境。在某些情况下，例如在 Docker
环境中，Python 对我们来说太大了。

这个项目用 Go 语言重新实现了 supervisord。编译后的 supervisord 非常适合没有安装
Python 的环境。

# 编译 supervisord

在编译 supervisord 之前，请确保你的环境中安装了 go-lang 1.11+。

要为 **Linux** 编译 supervisord，请运行以下命令：

1. go generate
2. GOOS=linux go build -tags release -a -ldflags "-linkmode external -extldflags
   -static" -o supervisord

要为 **Windows** 编译 supervisord，请在 `Windows PC` 上运行以下命令：

1. go mod tidy
2. go build -tags release -o supervisord.exe

# 运行 supervisord

生成 supervisord 二进制文件后，创建一个 supervisord 配置文件并像这样启动
supervisord：

```Shell
$ cat supervisor.conf
[program:test]
command = /your/program args
$ supervisord -c supervisor.conf
```

请注意，配置文件位置按以下顺序自动检测：

1. $CWD/supervisord.conf
2. $CWD/etc/supervisord.conf
3. /etc/supervisord.conf
4. /etc/supervisor/supervisord.conf (自 Supervisor 3.3.0 起)
5. ../etc/supervisord.conf (相对于可执行文件)
6. ../supervisord.conf (相对于可执行文件)

# 作为守护进程运行并启用 Web UI

在你的配置中添加 inet 接口：

```ini
[inet_http_server]
port=127.0.0.1:9001
```

然后运行

```shell
$ supervisord -c supervisor.conf -d
```

为了管理守护进程，你可以使用 `supervisord ctl`
子命令，可用的子命令有：`status`、`start`、`stop`、`shutdown`、`reload`。

```shell
$ supervisord ctl status
$ supervisord ctl status program-1 program-2...
$ supervisord ctl status group:*
$ supervisord ctl stop program-1 program-2...
$ supervisord ctl stop group:*
$ supervisord ctl stop all
$ supervisord ctl start program-1 program-2...
$ supervisord ctl start group:*
$ supervisord ctl start all
$ supervisord ctl shutdown
$ supervisord ctl reload
$ supervisord ctl signal <signal_name> <process_name> <process_name> ...
$ supervisord ctl signal all
$ supervisord ctl pid <process_name>
$ supervisord ctl fg <process_name>
```

请注意，`supervisor ctl` 子命令只有在 [inet_http_server] 中启用了 http
服务器并且正确设置了 **serverurl** 时才能正常工作。Unix
域套接字目前不支持此用途。

Serverurl 参数按以下顺序检测：

- 检查是否存在选项 -s 或 --serverurl，使用此 url
- 检查是否存在 -c 选项，并且 "supervisorctl" 部分中存在 "serverurl"，使用
  "supervisorctl" 部分中的 "serverurl"
- 检查自动检测的 supervisord.conf 文件位置中的 "supervisorctl" 部分是否定义了
  "serverurl"，如果定义了 - 使用找到的值
- 使用 http://localhost:9001

# 检查版本

命令 "version" 将显示当前 supervisord 二进制文件的版本。

```shell
$ supervisord version
```

# 支持的功能

## HTTP 服务器

HTTP 服务器可以通过 Unix 域套接字和 TCP 工作。基本身份验证也是可选和受支持的。

Unix 域套接字设置在 "unix_http_server" 部分。 TCP http 服务器设置在
"inet_http_server" 部分。

如果在配置文件中未同时设置 "inet_http_server" 和 "unix_http_server"，则不会启动
http 服务器。

## Supervisord 守护进程设置

在 "supervisord" 部分配置以下参数：

- **logfile**。将 supervisord 本身的日志放在哪里。
- **logfile_maxbytes**。在日志文件超过此长度后轮换日志文件。
- **logfile_backups**。保留的轮换日志文件数量。
- **loglevel**。日志详细程度，可以是 trace、debug、info、warning、error、fatal
  和 panic（根据所用模块的文档）。默认为 info。
- **pidfile**。包含当前 supervisord 实例进程 ID 的文件的完整路径。
- **minfds**。在 supervisord 启动时至少保留此数量的文件描述符。（Rlimit
  nofiles）。
- **minprocs**。在 supervisord 启动时至少保留此数量的进程资源。（Rlimit
  noproc）。
- **identifier**。此 supervisord
  实例的标识符。如果在同一命名空间中的同一台机器上运行多个
  supervisord，则这是必需的。

## 受监督程序设置

受监督程序设置在 [program:programName] 部分中配置，包括以下选项：

- **command**。要监督的命令。可以作为可执行文件的完整路径给出，也可以通过 PATH
  变量计算。命令行参数也应在此字符串中提供。
- **process_name**。进程名称
- **numprocs**。进程数量
- **numprocs_start**。？？
- **autostart**。受监督的命令是否应在 supervisord 启动时运行？默认为 **true**。
- **startsecs**。程序在启动后需要保持运行的总秒数，以认为启动成功（将进程从
  STARTING 状态移动到 RUNNING 状态）。设置为 0
  表示程序不需要保持运行任何特定时间。
- **startretries**。在放弃并将进程置于 FATAL 状态之前，supervisord
  在尝试启动程序时将允许的连续失败尝试次数。有关 FATAL
  状态的解释，请参阅进程状态。
- **autorestart**。如果受监督的命令死亡，自动重新运行。
- **exitcodes**。此程序的"预期"退出代码列表，与 autorestart 一起使用。如果
  autorestart 参数设置为 unexpected，并且程序以除 supervisor
  停止请求之外的任何方式退出，supervisord
  将在程序退出时使用此列表中未定义的退出代码重新启动程序。
- **stopsignal**。发送给命令以优雅停止它的信号。如果配置了多个
  stopsignal，在停止程序时，supervisor 将以 "stopwaitsecs"
  间隔逐个将信号发送给程序。如果在所有信号发送给程序后程序没有退出，supervisord
  将杀死该程序。
- **stopwaitsecs**。在将 SIGKILL
  发送给受监督命令以使其不优雅停止之前等待的时间量。
- **stdout_logfile**。受监督命令的 STDOUT
  应该重定向到哪里。（下面描述了特定值）。
- **stdout_logfile_maxbytes**。日志超过此大小后将轮换日志。
- **stdout_logfile_backups**。保留的轮换日志文件数量。
- **redirect_stderr**。STDERR 是否应该重定向到 STDOUT。
- **stderr_logfile**。受监督命令的 STDERR
  应该重定向到哪里。（下面描述了特定值）。
- **stderr_logfile_maxbytes**。日志超过此大小后将轮换日志。
- **stderr_logfile_backups**。保留的轮换日志文件数量。
- **environment**。要传递给受监督程序的 VARIABLE=value 列表。它比 `envFiles`
  具有更高的优先级。
- **envFiles**。要加载并传递给受监督程序的 .env 文件列表。
- **priority**。程序在启动和关闭顺序中的相对优先级
- **user**。在执行受监督命令之前切换到此 USER 或 USER:GROUP。
- **directory**。跳转到此路径并在那里执行受监督命令。
- **stopasgroup**。在停止列出了此程序的程序组时也停止此程序。
- **killasgroup**。在停止列出了此程序的程序组时也杀死此程序。
- **restartPause**。在停止受监督程序后再次启动它之前等待（至少）此秒数。
- **restart_when_binary_changed**。布尔值（false 或
  true）用于控制当受监督命令的可执行二进制文件更改时是否应重新启动。默认为
  false。
- **restart_cmd_when_binary_changed**。如果程序二进制文件本身更改，用于重新启动程序的命令。
- **restart_signal_when_binary_changed**。如果程序二进制文件更改，发送给程序用于重新启动的信号。
- **restart_directory_monitor**。要监控用于重新启动目的的路径。
- **restart_file_pattern**。如果 restart_directory_monitor
  下的文件更改且文件名匹配此模式，受监督的命令将被重新启动。
- **restart_cmd_when_file_changed**。如果 restart_directory_monitor 下具有模式
  restart_file_pattern 的任何监控文件发生更改，用于重新启动程序的命令。
- **restart_signal_when_file_changed**。如果 restart_directory_monitor
  下具有模式 restart_file_pattern
  的任何监控文件发生更改，将发送给程序的信号，例如 Nginx。
- **depends_on**。定义受监督命令启动依赖项。如果程序 A 依赖于程序 B、C，程序
  B、C 将在程序 A 之前启动。示例：

```ini
[program:A]
depends_on = B, C

[program:B]
...
[program:C]
...
```

## 为所有受监督程序设置默认参数

所有对于所有受监督程序相同的公共参数可以在 "program-default"
部分中定义一次，并在所有其他程序部分中省略。

在下面的示例中，VAR1 和 VAR2 环境变量适用于 test1 和 test2 受监督程序：

```ini
[program-default]
environment=VAR1="value1",VAR2="value2"
envFiles=global.env,prod.env

[program:test1]
...

[program:test2]
...
```

## 组

支持 "group" 部分，你可以设置 "programs" 项

## 事件

Supervisord 3.x 定义的事件部分受支持。现在它支持以下事件：

- 所有进程状态相关事件
- 进程通信事件
- 远程通信事件
- tick 相关事件
- 进程日志相关事件

## 日志

Supervisord 可以将受监督程序的 stdout 和 stderr（字段
stdout_logfile、stderr_logfile）重定向到：

- **/dev/null**。忽略日志 - 将其发送到 /dev/null。
- **/dev/stdout**。将日志写入 STDOUT。
- **/dev/stderr**。将日志写入 STDERR。
- **syslog**。将日志发送到本地 syslog 服务。
- **syslog @[protocol:]host[:port]**。将日志事件发送到远程 syslog
  服务器。协议必须是 "tcp" 或 "udp"，如果缺少，则假定为
  "udp"。如果缺少端口，对于 "udp" 协议，默认为 514，对于 "tcp" 协议，其值为
  6514。
- **文件名**。将日志写入指定的文件。

可以为 stdout_logfile 和 stderr_logfile 配置多个日志文件，以 ','
作为分隔符。例如：

```ini
stdout_logfile = test.log, /dev/stdout
```

### syslog 设置

如果将日志写入 syslog，可以设置以下附加参数：

```ini
syslog_facility=local0
syslog_tag=test
syslog_stdout_priority=info
syslog_stderr_priority=err
```

- **syslog_facility**，可以是以下之一（不区分大小写）：KERNEL、USER、MAIL、DAEMON、AUTH、SYSLOG、LPR、NEWS、UUCP、CRON、AUTHPRIV、FTP、LOCAL0~LOCAL7
- **syslog_stdout_priority**，可以是以下之一（不区分大小写）：EMERG、ALERT、CRIT、ERR、WARN、NOTICE、INFO、DEBUG
- **syslog_stderr_priority**，可以是以下之一（不区分大小写）：EMERG、ALERT、CRIT、ERR、WARN、NOTICE、INFO、DEBUG

# Web GUI

Supervisord 有内置的 Web GUI：你可以从 GUI
启动、停止和检查程序的状态。下图显示了默认的 Web GUI：

![alt text](go_supervisord_gui.png)

请注意，为了查看|使用 Web GUI，你应该在 /etc/supervisord.conf 中的
[inet_http_server]（和/或如果你更喜欢 unix 域套接字，则使用
[unix_http_server]）和 [supervisorctl] 中配置它：

```ini
[inet_http_server]
port=127.0.0.1:9001
;username=test1
;password=thepassword

[supervisorctl]
serverurl=http://127.0.0.1:9001
```

# 从 Docker 容器中使用

supervisord 在 Docker 镜像中编译，直接在另一个镜像中使用，来自 Docker Hub 版本。

```Dockerfile
FROM debian:latest
COPY --from=ochinchina/supervisord:latest /usr/local/bin/supervisord /usr/local/bin/supervisord
CMD ["/usr/local/bin/supervisord"]
```

# 与 Prometheus 集成

Prometheus 节点导出器支持的 supervisord 指标现在已集成到 supervisor
中。因此，无需部署额外的 node_exporter 来收集 supervisord
指标。要收集指标，必须在 "inet_http_server" 部分配置端口参数，并且指标服务器在
supervisor http 服务器的路径 /metrics 上启动。

例如，如果 "inet_http_server" 中的端口参数是
"127.0.0.1:9001"，那么指标服务器应该在 url "http://127.0.0.1:9001/metrics"
中访问

# 注册服务

在操作系统启动后自动启动 supervisord。在
[kardianos/service](https://github.com/kardianos/service) 查找支持的平台。

```Shell
# install
sudo supervisord service install -c full_path_to_conf_file
# uninstall
sudo supervisord service uninstall
# start
supervisord service start
# stop
supervisord service stop
```
