# 01 nginx 编译安装

官方下载源码包：https://nginx.org/download/nginx-1.24.0.tar.gz

解压进入，执行

```sh
./configure --prefix=/usr/local/nginx
```

如果遇到 PCRE 没有安装的，执行

```sh
sudo apt install libpcre3-dev
```

> 一般 libpcre3 包是默认安装的，如果没有，也一并安装

如果遇到 zlib 没有安装的，执行

```sh
sudo apt install zlib1g-dev
```

> 一般 zlib1g 包是默认安装的，如果没有，也一并安装

如果正常检查通过了，执行

```sh
make
```

```sh
sudo make install
```

没有问题的话，nginx 会安装在 `/usr/local/nginx` 下。

使用源码安装的 nginx 默认不会自动启动，需要运行

```sh
sudo /usr/local/nginx/sbin/nginx
```

手动启动。

## 补充一点关于服务的内容

### systemd 管理服务

```sh
systemctl start foo       # 启动服务
systemctl stop foo        # 停止服务
systemctl restart foo     # 重启服务
systemctl status foo      # 查看服务状态
systemctl enable foo      # 设置服务开机自启动
systemctl disable foo     # 设置服务不要开机自启动
systemctl is-enabled foo  # 查看服务是否是开机自启动
```

### 服务分类

使用二进制包安装的软件，其程序文件分散在系统的各个目录中，比如 `/bin`、`/var` 等。由二进制包安装的服务也可以由系统识别管理，通常是在 `/lib/systemd/system` 目录下以 `.service` 结尾的文件。这些服务可以使用 `systemctl` 管理。

使用源码包安装的软件，其程序文件不会分散在系统中，而是集中位于用户指定的位置（就像 Windows 里安装的软件一般在 `C:\Program Files\` 下），这个位置一般默认是 `/usr/local`，当然用户可以自己指定任意位置。源码包安装的服务不能被系统识别，默认不受 `systemctl` 管理，当然也不会自动启动了。

# 02 nginx 信号量

nginx 有一个文件用来保存当前 nginx 主进程号，如果是源码包安装的，默认是 `/usr/local/nginx/logs/nginx.pid`，如果是二进制包安装的，是 `/var/run/nginx.pid`。

该文件的位置可以通过 `nginx.conf` 文件的 `pid` 标记修改。源码包：`/usr/local/nginx/conf/nginx.conf`；二进制包：`/etc/nginx/nginx.conf`。

关于 nginx 的信号量控制，其官方文档：https://nginx.org/en/docs/control.html

> 以下默认在用二进制包安装的环境下进行。

查看 nginx 是否在运行

```text
julian@basoom:/etc/nginx$ ps aux | grep nginx
root         758  0.0  0.0  10548   988 ?        Ss   11:44   0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
nginx        759  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        761  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        762  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        763  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        764  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        765  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        766  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        767  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
julian      2770  0.0  0.0   6608  2292 pts/0    S+   22:57   0:00 grep --color=auto nginx
```

可以看到 nginx 有一个主进程和好几个工作进程。主进程不直接响应网页的请求，而是用来管理工作进程的；工作进程用来响应网页请求。

`kill` 命令的 man 手册是这样解释自己的：`kill - send a signal to a process`，所以 `kill` 不只是乱杀一通，它实际上的功能是给进程发送信号，这个信号老多，其中恰好包括强制终止进程的信号。`kill -l` 查看所有信号。

通过信号量控制 nginx，也就是用 `kill` 命令控制 nginx。

终止 nginx：

```text
julian@basoom:/etc/nginx$ sudo kill -INT 758

julian@basoom:/etc/nginx$ ps aux | grep nginx
julian      2857  0.0  0.0   6476  2424 pts/0    S+   23:04   0:00 grep --color=auto nginx
```

重新开启用 `sudo systemctl start nginx`。

`INT` 直接终止进程，`QUIT` 会等请求结束后再终止进程（优雅地终止进程）。

`HUP` 信号会用新的配置文件启动一个新的工作进程，然后优雅地关闭旧的工作进程。也就是说我们修改了配置文件之后不用重启整个 nginx 服务，只需要用 `HUP` 信号量就可以了。

比如我们修改一下配置文件 `/etc/nginx/conf.d/default.conf`，将

```text
location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
}
```

改为

```text
location / {
    root   /usr/share/nginx/html;
    index  ab.html index.html index.htm;
}
```

然后在 `/usr/share/nginx/html/` 目录下创建 `ab.html` 文件写入一些内容。

此时刷新网站还没有任何变化，执行 `sudo kill -HUP 2871` 后再刷新，就会看到 `ab.html` 文件的内容了。同时执行 `ps aux | grep nginx`，会看到 nginx 的所有工作进程号都跟之前不一样了，但是主进程的还一样。这也就是说，所有工作进程都重启了，但是主进程没有。

`USR1` 信号可以重开日志文件，也就是日志轮替。

默认 nginx 会把日志写在 `/var/log/nginx/access.log` 中，每刷新一次网站都会加一条记录。

当需要日志轮替时，先把 `access.log` 重命名，然后创建一个新的 `access.log` 文件，然后告诉 nginx 日志文件变了。如果不通知 nginx 的话，仅仅是改日志文件的名字并不会让 nginx 察觉到需要改变存储日志的位置，因为其底层通过 inode 号来识别文件，仅仅修改文件名并不会更改文件的 inode 号，因此哪怕改名了，nginx 依然是往这个 inode 号写日志。

通知 nginx 的方式即 `sudo kill -USR1 <pid>`。

# 03 nginx 虚拟主机配置
