# 基础

`/etc/ssh/ssh_config` 是 ssh 客户端配置文件。

`/etc/ssh/sshd_config` 是 ssh 服务端配置文件。

# 密钥管理

使用 ed25519 算法创建密钥

```sh
ssh-keygen -t ed25519
```

修改私钥的密码句（`-f` 后接私钥文件）

```sh
ssh-keygen -pf .ssh/id_ed25519
```

查看公钥指纹（`-f` 后接公钥文件）

```sh
ssh-keygen -lf .ssh/id_ed25519.pub
```

按照指定的算法生成指纹（默认是 SHA256 算法）

```sh
ssh-keygen -lf .ssh/id_ed25519.pub -E md5
```

以上创建和管理的都是客户端密钥，可以用于连接其他机器。

服务端密钥在安装 OpenSSH Server 的时候就已经创建了，在 `/etc/ssh` 目录下，以 `ssh_host_` 开头的几种算法的公私密钥对。

当客户端第一次发来 ssh 连接请求时，会要求确认指纹，此时可以使用如下命令在服务端查看指纹是否一致（默认使用 ed25519 算法的密钥，使用 SHA256 哈希算法）

```sh
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

# 服务端配置

打开配置文件（也要留意 `/etc/ssh/sshd_config.d` 目录下的其他配置）

```sh
sudo vim /etc/ssh/sshd_config
```

```text
# 禁止远程 root 登录
PermitRootLogin no
# 客户端使用公钥登录
PubkeyAuthentication yes
# 禁止密码登录
PasswordAuthentication no
```

如果要允许客户端通过密钥的方式登录，需要在 `~/.ssh/authorized_keys` 中加入客户端的 ssh 公钥（一行内容，复制粘贴进去即可）。

# 客户端配置

通常来说，使用形如 `ssh julian@192.168.113.15` 的命令就可以连接服务端了，但是每次都要输名字和地址，有点麻烦，因此可以定义一个 Host 块，保存这些信息。

创建 `~/.ssh/config` 文件，输入如下内容

```text
Host midgard
    HostName 192.168.113.15
    Port 22
    User julian
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

这就是一个 Host 块。

第一行定义这个块的别名，随便起。

第二行定义服务端的地址。

第三行定义服务端的 ssh 端口。

第四行定义要登录的用户名。

其实定义到这儿，就可以正常使用 `ssh midgard` 登录了，不过默认是密码登录，如果服务端禁用了密码登录，那么就登不上。

第五行定义要使用的客户端（即本地）私钥文件。

第六行定义只使用第五行定义的这个文件，而不去逐个尝试本地所有的 ssh 密钥文件。

接下来使用 `ssh midgard` 登录时就会用密钥登录了，如果私钥设置了密码句，那么也会要求输入该密码句。

注意，可以用密钥登录的前提是在服务端的 `~/.ssh/authorized_keys` 文件中添加了客户端中与第五行私钥文件对应的公钥文件内容。否则的话还是用密码登录的。

# Windows 相关

Windows 作为客户端的配置跟前面说的都差不多，`ssh_keygen` 创建和管理密钥，密钥和配置文件都在 `~\.ssh` 目录下。现在 Windows 一般都默认安装 OpenSSH 客户端了，所以不需要多做什么。

但是 Windows 默认没有安装 OpenSSH Server，因此默认无法作为服务端让其他机器连接。

以 Windows 11 为例，在设置-系统-可选功能中，在添加可选功能行点击查看功能，找到 OpenSSH 服务器打勾并安装。安装后即可拥有 ssh 服务端功能。（其他系统版本也是找可选功能）

安装后服务端的密钥和配置文件在 `C:\ProgramData\ssh` 目录下，修改需要管理员权限。

其中 `C:\ProgramData\ssh\sshd_config` 的配置跟前面说的基本一致。

如果要允许客户端使用密钥登录 Windows 上的 **管理员组的用户**（一般装机的第一个用户都是管理员组的用户），则需要在 `C:\ProgramData\ssh\administrators_authorized_keys` 文件中添加客户端的公钥。

如果要允许客户端使用密钥登录 Windows 上的 **普通用户**（非管理员组的用户），则需要在 `~\.ssh\authorized_keys` 文件中添加客户端的公钥。

> 另见 [Get started with OpenSSH for Windows | Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui#install-openssh-for-windows) 中的 Next steps。

# MacOS 相关

重启 SSH 服务

```sh
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```

# 批量分发密钥

上面核对公钥指纹，将自己的公钥添加到服务器的许可密钥文件等操作，如果只有一台服务器，那手动操作还可以，如果有好几台，好几十台等等，那手动操作就不现实了。

`ssh-keyscan` 可以获取特定 IP 的指纹，可以取代我们第一次连接服务器时要求主动核对公钥指纹的步骤。

```sh
ssh-keyscan 192.168.113.15  # 会把扫描到的指纹输出到屏幕
ssh-keyscan 192.168.113.15 >> ~/.ssh/known_hosts  # 把扫描到的指纹原文写入文件（windows的方式）
ssh-keyscan -H 192.168.113.15 >> ~/.ssh/known_hosts  # 把扫描到的指纹加密写入文件（Linux的方式）
```

`ssh-copy-id` 可以将自己的公钥添加到对方服务器的许可文件中。

```sh
ssh-copy-id -i ~/.ssh/id_ed25519.pub julian@192.168.113.4
```

`-i` 指定要添加哪个公钥文件，后面是主机 IP 和用户名。公钥会添加到该用户的家目录下的 `.ssh/authorized_keys` 文件中。

这一步会要求输入该用户的登录密码，依然是交互输入，可以使用 sshpass 解决。

sshpass 可能默认没有安装，可以使用 apt 安装。`-p` 参数可以指定密码，假设 `123456` 是用户登录密码，那么上面命令可以改写为

```sh
sshpass -p 123456 ssh-copy-id -i ~/.ssh/id_ed25519.pub julian@192.168.113.4
```

Windows 并没有 `ssh-copy-id` 命令，可以使用如下代替

```sh
type ~/.ssh/id_ed25519.pub | ssh julian@192.168.113.4 "cat >> ~/.ssh/authorized_keys"
```
