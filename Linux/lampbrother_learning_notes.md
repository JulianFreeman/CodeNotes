# 第四讲 Linux 常用命令

## 4.1 文件处理命令

### 4.1.1 命令格式与目录处理命令 `ls`

使用 `ls -l /home/julian` 显示的是该目录下文件和目录的信息，要想查看该目录本身的信息，可以用 `-d` 参数。

```text
julian@ubuntu2204:~$ ls -l /home/julian
total 36
drwxr-xr-x 3 julian julian 4096 Nov 24 13:26 Desktop
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Documents
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Downloads
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Music
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Pictures
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Public
drwx------ 3 julian julian 4096 Nov 24 11:54 snap
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Templates
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Videos
```

```text
julian@ubuntu2204:~$ ls -ld /home/julian
drwxr-x--- 14 julian julian 4096 Nov 24 13:31 /home/julian
```

### 4.1.2 目录处理命令

#### `mkdir`

`mkdir` 的 `-p` 参数可以 **递归** 创建目录。

```text
julian@ubuntu2204:~/Documents$ mkdir -p ./abc/def
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    └── def

2 directories, 0 files
```

`mkdir` 可以同时创建 **多个** 目录。

```text
julian@ubuntu2204:~/Documents$ mkdir ./abc/ghi ./abc/jkl
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    ├── ghi
    └── jkl

4 directories, 0 files
```

#### `cp`

`cp` 可以同时复制 **多个** 文件，只要把目标目录放在最后就好。

```text
julian@ubuntu2204:~/Documents$ cp /etc/hosts /etc/hosts.allow ./abc/
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    ├── ghi
    ├── hosts
    ├── hosts.allow
    └── jkl

4 directories, 2 files
```

`cp` 默认只能复制文件，但是加 `-r` 参数可以 **递归** 复制目录。

```text
julian@ubuntu2204:~/Documents$ cp -r /etc/environment.d/ ./abc/jkl/
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    ├── ghi
    ├── hosts
    ├── hosts.allow
    └── jkl
        └── environment.d
            ├── 90atk-adaptor.conf
            └── 90qt-a11y.conf

5 directories, 4 files
```

当然同时复制文件和目录也是可以的，只要加上 `-r` 而且最后是目标目录就好。

```text
julian@ubuntu2204:~/Documents$ cp -r ./abc/jkl/environment.d/ ./abc/hosts ./abc/def/
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    │   ├── environment.d
    │   │   ├── 90atk-adaptor.conf
    │   │   └── 90qt-a11y.conf
    │   └── hosts
    ├── ghi
    ├── hosts
    ├── hosts.allow
    └── jkl
        └── environment.d
            ├── 90atk-adaptor.conf
            └── 90qt-a11y.conf

6 directories, 7 files
```

`cp` 复制时默认不复制源文件属性，加 `-p` 参数可以保留源文件属性（比如时间）。

```text
julian@ubuntu2204:~/Documents$ ls -l /etc/hosts
-rw-r--r-- 1 root root 225 Nov 24 11:47 /etc/hosts

julian@ubuntu2204:~/Documents$ ls -l ./abc/hosts
-rw-r--r-- 1 julian julian 225 Nov 24 14:08 ./abc/hosts

julian@ubuntu2204:~/Documents$ cp -p /etc/hosts ./abc/ghi/

julian@ubuntu2204:~/Documents$ ls -l ./abc/ghi/hosts 
-rw-r--r-- 1 julian julian 225 Nov 24 11:47 ./abc/ghi/hosts
```

`cp` 在复制文件或目录的同时也可以 **重命名**，只要 **目标路径不存在** 即可。

```text
julian@ubuntu2204:~/Documents$ cp /etc/hosts ./abc/myhosts
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    ├── ghi
    └── myhosts

3 directories, 1 file
```

```text
julian@ubuntu2204:~/Documents$ cp -r /etc/environment.d/ ./abc/env.d
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    ├── env.d
    │   ├── 90atk-adaptor.conf
    │   └── 90qt-a11y.conf
    ├── ghi
    └── myhosts

4 directories, 3 files
```

但是重命名只能在复制 **单个** 文件或者目录时进行，如果复制多个文件或目录，目标路径 **必须是一个存在的目录**。

#### 关于 `cp` 的一些使用情况说明

参数中涉及到目录时，最后加不加 `/` 应该影响不大，因为同一目录下的子文件和子目录之间也不能重名，所以程序能知道哪个是目录，哪个是文件。

当路径最后有 `/` 的时候，程序会把该路径当目录处理，如果找不到这个目录，哪怕能找到同名文件，也会报错，因为文件不是目录。

当路径最后没有 `/` 的时候，程序会根据情况，找到发现是目录，就当目录处理，找到发现是文件，就当文件处理，啥也没找到，就报错。这里目录和文件没有先后，因为不可能既有目录又有文件，前面说了，文件和目录也不能重名。

所以为了严谨，我们可以把目录路径后加 `/` 以作标识。

当使用 `cp` 复制单个路径时，

- 如果源路径是文件，
  - 如果目标路径是文件，
    - 如果目标文件不存在：复制并重命名
    - 如果目标文件存在：覆盖复制并可能重命名（取决于源和目标是不是重名）
  - 如果目标路径是目录，
    - 如果目标目录不存在：报错
    - 如果目标目录存在：复制到该目录下（不关心目标目录与源文件是否重名）
- 如果源路径是目录，
  - 如果目标路径是文件（这里指路径最后没有加 `/`），
    - 如果目标文件不存在：复制并重命名（目标路径当作目录处理）
    - 如果目标文件存在：报错（文件和目录不能重名）
  - 如果目标路径是目录：
    - 如果目标目录不存在：复制并重命名
    - 如果目标目录存在：复制到该目录下

当使用 `cp` 复制多个路径时，

- 如果目标路径是一个存在的目录：复制到该目录下
- 否则，
  - 如果目标路径是一个存在的文件：报错（文件不是目录）
  - 如果目标路径不存在：报错（必须存在）

所以，要判断一个 `cp` 命令会不会导致重命名，

1. 看源路径是不是只有一个
2. 看目标路径是不是不存在

不考虑报错且源路径只有一个的情况下，目标路径不存在 **一定会重命名**，存在则 **只有** 一种情况会重命名，也就是 **源和目标都存在且都是文件时**，也就是发生 **覆盖** 时。

#### `mv`

`mv` 命令可以理解为 `cp` 之后把源文件或目录也删掉，因此其对复制或者重命名的处理跟 `cp` 是一样的。只不过 `cp` 要处理目录的话得加 `-r` 参数，但 `mv` 不需要加参数就可以处理目录。

其实，把上面的分层结构中的“复制”改为“移动”就符合 `mv` 的情况了，无论源路径是单个还是多个，都符合。

### 4.1.3 文件处理命令

#### `cat`

`cat` 就是打印 **所有** 文件内容，没法分页，也没法打印局部。其 `-n` 参数指的是打印时 **添加行号**。

直接用 `batcat`（`sudo apt install bat`）代替了。

#### 命令行查看文档的一些操作

> `more` 好像也可以往上翻页。

使用 `less` 或者 `man` 之类的命令在命令行查看帮助文档时，

- 敲 `/`，输入要搜索的文本，回车，文档内能匹配的文本会高亮，此时敲 `n` 可以查看下一个匹配，敲 `N` 可以查看上一个匹配（`Shift+n`）

> `vim` 也可以，不过这个可以单独讲了。

### 4.1.4 链接命令

`ln -s [源路径] [目标路径]` 可以创建软链接。

如果目标路径就在当前的工作目录中，则源路径是相对路径或者绝对路径都可以。

如果目标路径不在当前的工作目录中，则源路径必须是绝对路径，否则软链接不生效。

`ln` 创建硬链接则好像没有这个要求。不过硬链接只能作用于文件，且不能跨分区。

## 4.2 权限管理命令

### 4.2.1 权限管理命令 `chmod`

同时进行多种授权时用逗号隔开：

```text
julian@ubuntu2204:~/Documents$ ls -l ./abc/issue.cp 
-rw-r--r-- 2 julian julian 26 Nov 24 20:39 ./abc/issue.cp

julian@ubuntu2204:~/Documents$ chmod g+w,o-r ./abc/issue.cp 

julian@ubuntu2204:~/Documents$ ls -l ./abc/issue.cp 
-rw-rw---- 2 julian julian 26 Nov 24 20:39 ./abc/issue.cp
```

授权按照先后顺序进行，比如上例，`g+w,o-r` 表示先给所属组加写权限，再给其他人减读权限。这样如果权限的设置前后有冲突，以在后的为准。

当然，用数字就没这些冲突了：

```text
julian@ubuntu2204:~/Documents$ ls -l ./abc/issue.cp 
-rw-rw---- 2 julian julian 26 Nov 24 20:39 ./abc/issue.cp

julian@ubuntu2204:~/Documents$ chmod 644 ./abc/issue.cp 

julian@ubuntu2204:~/Documents$ ls -l ./abc/issue.cp 
-rw-r--r-- 2 julian julian 26 Nov 24 20:39 ./abc/issue.cp
```

文件或目录的权限只有其 **所有者和 root 用户** 可以修改。

#### 权限对文件和目录的含义

|权限|对文件|对目录|
|--:|--|--|
|读权限 `r`|可以查看文件内容|可以列出目录内容|
|写权限 `w`|可以修改文件内容|可以在目录创建、删除（也就是修改目录里的内容）|
|执行权限 `x`|可以执行文件|可以进入目录|

可能把目录理解为记录了其内所有文件和目录名称的名单文件，就好理解对目录的读写权限的含义了。读就是能看到名单文件的内容，也就是目录里有什么文件和目录，写就是修改这个名单文件，也就是修改目录里的文件和目录，即创建或删除。

**目录的读和执行权限都是同时出现的**，否则不能正常使用。

### 4.2.2 其他权限管理命令

#### `chown`

只有 root 用户可以修改文件的所有者，所有者自己都不行。

`chown` 文档原话：

- `chown` 修改给定的每一个文件的所有者和/或所属组。
- 如果指定了用户名，这个用户名就被修改为文件的所有者，文件的所属组不变。

```text
root@ubuntu2204:/tmp/abc# ls -dl def/
drwxrwxr-x 2 karl karl 4096 Nov 24 23:03 def/

root@ubuntu2204:/tmp/abc# chown julian def/

root@ubuntu2204:/tmp/abc# ls -dl def/
drwxrwxr-x 2 julian karl 4096 Nov 24 23:03 def/
```

- 如果用户名后面跟着一个冒号和一个组名，中间没有空格，那么文件的所属组也会改变。

```text
root@ubuntu2204:/tmp/abc# ls -dl ghi/
drwxrwxr-x 2 karl karl 4096 Nov 24 23:13 ghi/

root@ubuntu2204:/tmp/abc# chown julian:julian ghi/

root@ubuntu2204:/tmp/abc# ls -dl ghi/
drwxrwxr-x 2 julian julian 4096 Nov 24 23:13 ghi/
```

- 如果用户名后面有冒号但没有指定组名，那么文件的所有者改为该用户，所属组改为该用户的登录组（login group）。

```text
root@ubuntu2204:/tmp/abc# ls -dl jkl/
drwxrwxr-x 2 karl karl 4096 Nov 24 23:13 jkl/

root@ubuntu2204:/tmp/abc# chown julian: jkl/

root@ubuntu2204:/tmp/abc# ls -dl jkl/
drwxrwxr-x 2 julian julian 4096 Nov 24 23:13 jkl/
```

- 如果有冒号和组名，但没有指定用户名，那么只有文件的所属组被改变，这种情况下 `chown` 就跟 `chgrp` 作用一样了。

```text
root@ubuntu2204:/tmp/abc# ls -dl mno/
drwxrwxr-x 2 karl karl 4096 Nov 24 23:13 mno/

root@ubuntu2204:/tmp/abc# chown :julian mno/

root@ubuntu2204:/tmp/abc# ls -dl mno/
drwxrwxr-x 2 karl julian 4096 Nov 24 23:13 mno/
```

- 如果只给了一个冒号，或者啥也没给，那么什么都不会改变。

#### `chgrp`

修改所属组就很简单了，当然也只有 root 用户有权限修改。

```text
root@ubuntu2204:/tmp/abc# ls -dl mno/
drwxrwxr-x 2 karl julian 4096 Nov 24 23:13 mno/

root@ubuntu2204:/tmp/abc# chgrp karl mno/

root@ubuntu2204:/tmp/abc# ls -dl mno/
drwxrwxr-x 2 karl karl 4096 Nov 24 23:13 mno/
```

#### `umask`

`umask` 设定的是创建文件时的初始权限。加 `-S` 参数可以以更直观的方式显示。

```text
julian@ubuntu2204:~/Documents$ umask -S
u=rwx,g=rwx,o=rx
```

这意味着创建新的目录，其初始权限为 `rwxrwxr-x`。因为 Linux 规定缺省权限创建的新文件不能有执行权限，因此默认新文件的初始权限为 `rw-rw-r--`。

## 4.3 文件搜索命令

### 4.3.1 文件搜索命令 `find`

#### 按名称查找

名称全匹配：

```text
root@ubuntu2204:~# find /etc -name init
/etc/init
/etc/apparmor/init
```

名称包含匹配：

```text
root@ubuntu2204:~# find /etc -name *init*
/etc/init
/etc/security/namespace.init
/etc/apparmor/init
/etc/gdb/gdbinit
/etc/gdb/gdbinit.d
/etc/X11/xinit
/etc/X11/xinit/xinitrc
...
```

名称不区分大小写：

```text
root@ubuntu2204:~# find /etc -iname init
/etc/init
/etc/apparmor/init
/etc/gdm3/Init
```

#### 按大小查找

`-size` 后的参数的单位是块，1 个块是 512B，因此 204800 个块是 204800 * 512B = 100MB 大小，加号代表大于，减号代表小于

```text
root@ubuntu2204:~# find / -size +204800
/snap/firefox/2987/usr/lib/firefox/libxul.so
/snap/gnome-42-2204/141/usr/lib/x86_64-linux-gnu/libLLVM-15.so.1
/snap/gnome-42-2204/120/usr/lib/x86_64-linux-gnu/libLLVM-15.so.1
...

root@ubuntu2204:~# ls -lh /snap/firefox/2987/usr/lib/firefox/libxul.so
-rwxr-xr-x 1 root root 138M Aug  7 06:31 /snap/firefox/2987/usr/lib/firefox/libxul.so
```

#### 按用户或组查找

`-user` 按照所有者查找，`-group` 按照所属组查找。

#### 按时间查找

`-amin` 访问文件时间（access），`-cmin` 修改文件属性时间（change），`-mmin` 修改文件内容时间（modify），后面的数字单位都是分钟，加号代表超过这个时间，减号代表在这个时间之内

下例表示 `/etc` 目录中 100 分钟之内修改过文件属性的文件

```text
root@ubuntu2204:~# find /etc -cmin -100
/etc/cups
/etc/cups/subscriptions.conf
/etc/cups/subscriptions.conf.O
```

#### 结合多个条件查找

`-a` 表示 and，即多个条件同时满足

下例表示查找大小大于 80MB 同时小于 100MB 的文件

```text
root@ubuntu2204:~# find / -size +163840 -a -size -204800
/snap/gnome-3-38-2004/143/usr/lib/x86_64-linux-gnu/libLLVM-12.so.1
...

root@ubuntu2204:~# ls -hl /snap/gnome-3-38-2004/143/usr/lib/x86_64-linux-gnu/libLLVM-12.so.1
-rw-r--r-- 1 root root 88M Mar  8  2022 /snap/gnome-3-38-2004/143/usr/lib/x86_64-linux-gnu/libLLVM-12.so.1
```

`-o` 表示 or，即多个条件满足一个即可

#### 按类型查找

`-type` 可以接 `f`（文件），`d`（目录），`l`（软链接）

下例表示查找名称为 `init` 开头的目录

```text
root@ubuntu2204:~# find /etc -name init* -a -type d
/etc/init
/etc/apparmor/init
/etc/init.d
/etc/initramfs-tools
/etc/initramfs-tools/scripts/init-premount
/etc/initramfs-tools/scripts/init-bottom
/etc/initramfs-tools/scripts/init-top
```

#### 查找后执行

`-exec` 后接要执行的命令，前面查找的结果用 `{}` 替换，最后加一个 `\;`

下例查找 `/etc` 目录下所有以 `init` 开头的目录后执行 `ls -dl` 列举了它们的属性

```text
root@ubuntu2204:~# find /etc -name init* -a -type d -exec ls -dl {} \;
drwxr-xr-x 2 root root 4096 Nov 24 11:49 /etc/init
drwxr-xr-x 3 root root 4096 Aug  7 19:53 /etc/apparmor/init
drwxr-xr-x 2 root root 4096 Nov 24 12:22 /etc/init.d
drwxr-xr-x 5 root root 4096 Nov 24 12:22 /etc/initramfs-tools
drwxr-xr-x 2 root root 4096 Jun 14 03:54 /etc/initramfs-tools/scripts/init-premount
drwxr-xr-x 2 root root 4096 Jun 14 03:54 /etc/initramfs-tools/scripts/init-bottom
drwxr-xr-x 2 root root 4096 Jun 14 03:54 /etc/initramfs-tools/scripts/init-top
```

将 `-exec` 替换为 `-ok` 会在对每一个查找到的结果执行命令前进行确认，通常如果查找后执行的是删除命令，`-ok` 可能会有用。

#### 按 i 结点查找

`-inum` 可以按照文件结点查找。

如果有一个名称很怪的文件，不知道怎么手打出来，此时可以先查看其 i 结点号，然后用 `find` 找到后再执行某些操作

```text
root@ubuntu2204:/tmp/abc# ls -i
655376 'weird name file'

root@ubuntu2204:/tmp/abc# find . -inum 655376 -exec cat {} \;
pretend this is a file with a weird name
```

同时使用 i 结点查找也能找到一块分区内某一个文件的所有硬链接。

### 4.3.2 其他搜索命令

#### `locate`

在 Ubuntu 22.04.3 中这个命令没有默认安装，需要安装 `plocate` 包。

使用前先用 `updatedb` 命令更新索引库，默认的索引文件在 `/var/lib/plocate/plocate.db`。

`locate` 区分大小写，但并不是全匹配的

```text
julian@ubuntu2204:~/temp$ locate to_be
/home/julian/temp/to_be_or_not
```

`-i` 可以不区分大小写

```text
julian@ubuntu2204:~/temp$ locate -i to_be
/home/julian/temp/TO_BE_OR_NOT
/home/julian/temp/to_be_or_not
```

`locate` 默认匹配的是全路径，也就是说，只要一个目录名符合条件，该目录下的所有文件和目录都符合条件，因为它们的路径中一定会包含这个符合条件的父目录

```text
julian@ubuntu2204:~/temp$ tree
.
├── to_be
│   ├── bill.t
│   └── shakes.txt
├── to_be_or_not
└── TO_BE_OR_NOT

1 directory, 4 files
julian@ubuntu2204:~/temp$ locate to_be
/home/julian/temp/to_be
/home/julian/temp/to_be_or_not
/home/julian/temp/to_be/bill.t
/home/julian/temp/to_be/shakes.txt
```

`-b` 参数可以设置为只匹配文件名或目录名

```text
julian@ubuntu2204:~/temp$ locate -b to_be
/home/julian/temp/to_be
/home/julian/temp/to_be_or_not
```

`-c` 参数可以只计数而不罗列所有项

```text
julian@ubuntu2204:~/temp$ locate -bc init
3070
```

有些路径 `locate` 并不会索引，比如 `/tmp` 目录。

#### `which`

搜索命令文件的绝对路径

```text
julian@ubuntu2204:~/temp$ which ls
/usr/bin/ls
```

#### `whereis`

搜素命令文件的绝对路径和该命令文档的绝对路径

```text
julian@ubuntu2204:~/temp$ whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

#### `grep`

文件内容查找命令

```text
julian@ubuntu2204:~/temp$ grep "to be" to_be_or_not 
So this is a to be or not to be question
so to be or not to be ?

julian@ubuntu2204:~/temp$ grep so to_be_or_not 
so to be or not to be ?
```

同样默认区分大小写，加 `-i` 不区分

```text
julian@ubuntu2204:~/temp$ grep -i so to_be_or_not 
So this is a to be or not to be question
so to be or not to be ?
```

`-v` 反选

```text
julian@ubuntu2204:~/temp$ grep -vi so to_be_or_not 
yes you're right.
nah, this is only a text, don't be serious
```

## 4.4 帮助命令

`man` 命令不仅可以看命令的帮助文档，也可以看配置文件的帮助文档。比如 `man services` 会打开 `/etc/services` 配置文件的帮助文档。

查看 `passwd` 会发现它即是一个命令，也是一个配置文件

```text
julian@ubuntu2204:~/temp$ whereis passwd
passwd: /usr/bin/passwd /etc/passwd /usr/share/man/man1/passwd.1.gz /usr/share/man/man1/passwd.1ssl.gz /usr/share/man/man5/passwd.5.gz
```

此时调用 `man passwd` 默认显示的是命令的帮助文档，如果想看配置文件的帮助文档，可以用 `man 5 passwd`。

Linux 的帮助文档中 `1` 表示命令的帮助文档，`5` 表示配置文件的帮助文档。

`whatis` 可以查看命令和配置文件的 *一行* 帮助信息。

`apropos` 可以检索帮助文档

```text
julian@ubuntu2204:~/temp$ whatis apropos
apropos (1)          - search the manual page names and descriptions

julian@ubuntu2204:~/temp$ apropos passwd
chgpasswd (8)        - update group passwords in batch mode
chpasswd (8)         - update passwords in batch mode
fgetpwent_r (3)      - get passwd file entry reentrantly
getpwent_r (3)       - get passwd file entry reentrantly
gpasswd (1)          - administer /etc/group and /etc/gshadow
grub-mkpasswd-pbkdf2 (1) - generate hashed password for GRUB
openssl-passwd (1ssl) - compute password hashes
pam_localuser (8)    - require users to be listed in /etc/passwd
passwd (1)           - change user password
passwd (1ssl)        - OpenSSL application commands
passwd (5)           - the password file
passwd2des (3)       - RFS password encryption
update-passwd (8)    - safely update /etc/passwd, /etc/shadow and /etc/group
```

有些 Shell 内置命令无法用 `man` 查看帮助文档，比如 `cd`。

用 `help` 可以查看内置命令的帮助文档。比如 `help cd`。

比如 `help if` 也是可以的。

## 4.5 用户管理命令

`useradd` 是一个更底层的命令，常规添加用户应该用 `adduser`。

`adduser` 可以创建普通用户，创建系统用户，创建普通组，创建系统组，把一个存在的用户添加到一个存在的组。

`passwd` 可以修改用户的密码。普通用户只能修改自己的密码，而且还要符合复杂度的要求，但是 root 用户可以修改所有人的密码，且不用遵守复杂度要求。

`who` 可以查看现在有多少个用户在登录。

`w` 可以查看更多信息。其展示的信息都是什么意思，可以查看帮助文档 `man w`。

## 4.6 压缩解压命令

### `gzip`

后缀一般为 .gz 的压缩文件。关于 `gzip` 的命令帮助可以查看 `gzip --help`。

压缩文件

```text
julian@ubuntu2204:~/temp$ ls
lorem

julian@ubuntu2204:~/temp$ gzip lorem 

julian@ubuntu2204:~/temp$ ls
lorem.gz
```

加 `-k` 保留源文件

```text
julian@ubuntu2204:~/temp$ ls
lorem

julian@ubuntu2204:~/temp$ gzip -k lorem 

julian@ubuntu2204:~/temp$ ls
lorem  lorem.gz
```

`-d` 解压文件（与 `gunzip` 相同），加 `-k` 也是保留源文件

```text
julian@ubuntu2204:~/temp$ ls
lorem.gz

julian@ubuntu2204:~/temp$ gzip -dk lorem.gz 

julian@ubuntu2204:~/temp$ ls
lorem  lorem.gz
```

`-l` 可以在解压文件的情况下查看压缩文件内的内容。

已经以 .gz 结尾的文件不能被压缩（哪怕并不是 gzip 压缩文件），不是以 .gz 结尾的文件不能被解压（哪怕实际是 gzip 压缩文件），但能用 `-l` 查看其内容。

压缩和解压貌似都不能重命名。

gzip 不能压缩目录。

只能压缩单文件，哪怕指定了多个文件名，也是分别压缩的。

### `tar`

`tar` 是一个打包命令，可以把多个文件或目录打包成一个文件。打包的同时，也可以压缩。

`-c` 创建（create）包，`-f` 指定生成的文件名，必须要有

```text
julian@ubuntu2204:~/temp$ tree
.
├── elit
├── ipsum
│   └── lorem
└── sanc
    └── manga

2 directories, 3 files

julian@ubuntu2204:~/temp$ tar -cf my.tar ipsum/ sanc/ elit 

julian@ubuntu2204:~/temp$ ls
elit  ipsum  my.tar  sanc
```

`-t` 可以查看包的内容，`-v` 可以列举更详细的信息，`-f` 还是指定文件名

```text
julian@ubuntu2204:~/temp$ tar -tvf my.tar 
drwxrwxr-x julian/julian     0 2023-11-25 23:30 ipsum/
-rw-rw-r-- julian/julian  2457 2023-11-25 20:44 ipsum/lorem
drwxrwxr-x julian/julian     0 2023-11-25 23:34 sanc/
-rw-rw-r-- julian/julian  1712 2023-11-25 23:34 sanc/manga
-rw-rw-r-- julian/julian  3347 2023-11-25 23:35 elit
```

`-x` 解包，`-f` 还是指定文件名

```text
julian@ubuntu2204:~/temp/abc$ ls
my.tar

julian@ubuntu2204:~/temp/abc$ tar -xf my.tar 

julian@ubuntu2204:~/temp/abc$ tree
.
├── elit
├── ipsum
│   └── lorem
├── my.tar
└── sanc
    └── manga

2 directories, 4 files
```

解包时不能指定目标路径。

解包时可以只指定部分内容解包（下例只解了包中的一个目录）

```text
julian@ubuntu2204:~/temp/abc$ ls
my.tar

julian@ubuntu2204:~/temp/abc$ tar -xf my.tar ipsum/

julian@ubuntu2204:~/temp/abc$ ls
ipsum  my.tar
julian@ubuntu2204:~/temp/abc$ tree
.
├── ipsum
│   └── lorem
└── my.tar

1 directory, 2 files
```

删除包中的内容（添加的话把 `--delete` 改为 `--append`）

```text
julian@ubuntu2204:~/temp/abc$ tar -f my.tar --delete elit

julian@ubuntu2204:~/temp/abc$ tar -tvf my.tar 
drwxrwxr-x julian/julian     0 2023-11-25 23:30 ipsum/
-rw-rw-r-- julian/julian  2457 2023-11-25 20:44 ipsum/lorem
drwxrwxr-x julian/julian     0 2023-11-25 23:34 sanc/
-rw-rw-r-- julian/julian  1712 2023-11-25 23:34 sanc/manga
```

打包时使用 gzip 压缩

```text
julian@ubuntu2204:~/temp$ tar -cf my.tar.gz elit ipsum/ sanc/ --gzip

julian@ubuntu2204:~/temp$ ls
abc  elit  ipsum  my.tar.gz  sanc
```

当然使用 `tar -zcf` 也是可以的。

`tar -xf` 貌似也可以识别出被压缩过的包，并自动解压缩。

```text
julian@ubuntu2204:~/temp/abc$ ls
my.tar.gz

julian@ubuntu2204:~/temp/abc$ tar -xf my.tar.gz 

julian@ubuntu2204:~/temp/abc$ tree
.
├── elit
├── ipsum
│   └── lorem
├── my.tar.gz
└── sanc
    └── manga

2 directories, 4 files
```

选择其他压缩格式

```text
julian@ubuntu2204:~/temp/abc$ tar -cf my.tar.bz elit ipsum/ sanc/ --bzip2

julian@ubuntu2204:~/temp/abc$ ls
elit  ipsum  my.tar.bz  sanc

julian@ubuntu2204:~/temp/abc$ file my.tar.bz 
my.tar.bz: bzip2 compressed data, block size = 900k
```

```text
julian@ubuntu2204:~/temp/abc$ tar -cf my.tar.xz elit ipsum/ sanc/ --xz

julian@ubuntu2204:~/temp/abc$ ls
elit  ipsum  my.tar.xz  sanc

julian@ubuntu2204:~/temp/abc$ file my.tar.xz 
my.tar.xz: XZ compressed data, checksum CRC64
```

### `zip`

压缩文件（不会删除源文件）

```text
julian@ubuntu2204:~/temp/abc$ ls
elit  ipsum  sanc

julian@ubuntu2204:~/temp/abc$ zip elit.zip elit 
  adding: elit (deflated 64%)

julian@ubuntu2204:~/temp/abc$ ls
elit  elit.zip  ipsum  sanc
```

加 `-r` 可以压缩目录

```text
julian@ubuntu2204:~/temp/abc$ ls -F
elit  ipsum/  sanc/

julian@ubuntu2204:~/temp/abc$ zip -r my.zip elit ipsum/ sanc/
  adding: elit (deflated 64%)
  adding: ipsum/ (stored 0%)
  adding: ipsum/lorem (deflated 62%)
  adding: sanc/ (stored 0%)
  adding: sanc/manga (deflated 59%)

julian@ubuntu2204:~/temp/abc$ ls
elit  ipsum  my.zip  sanc
```

### `unzip`

`-l` 可以列举 zip 压缩包的内容，加 `-v` 可以显示更详细的信息

```text
julian@ubuntu2204:~/temp/abc$ unzip -l my.zip 
Archive:  my.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
     3347  2023-11-25 23:35   elit
        0  2023-11-25 23:30   ipsum/
     2457  2023-11-25 20:44   ipsum/lorem
        0  2023-11-25 23:34   sanc/
     1712  2023-11-25 23:34   sanc/manga
---------                     -------
     7516                     5 files
```

`-d` 可以指定解压目录

```text
julian@ubuntu2204:~/temp/abc$ tree
.
├── def
└── my.zip

1 directory, 1 file

julian@ubuntu2204:~/temp/abc$ unzip my.zip -d def/
Archive:  my.zip
  inflating: def/elit                
   creating: def/ipsum/
  inflating: def/ipsum/lorem         
   creating: def/sanc/
  inflating: def/sanc/manga          

julian@ubuntu2204:~/temp/abc$ tree
.
├── def
│   ├── elit
│   ├── ipsum
│   │   └── lorem
│   └── sanc
│       └── manga
└── my.zip

3 directories, 4 files
```

### `bzip2`

bzip2 是 gzip 的升级版，文件后缀默认是 .bz2，更适合压缩大文件。

其操作方式及特点跟 gzip 很相似，比如

- 默认不保留源文件
- 不能压缩目录
- 只能压缩单文件
- 已经以 .bz2 结尾的文件不能再压缩（哪怕实际不是 bzip2 压缩文件）

但也有些地方不同，比如

- 没有 `-l` 列举压缩文件内容。
- 不以 .bz2 结尾但实际是 bzip2 压缩包的文件可以解压

### 总结

总结来说，`zip` 是 Windows 和 Linux 通用的，有些相似之处，可以同时处理目录和文件。但是 `gzip` 和 `bzip2` 都只是压缩命令，不能打包，而 `tar` 负责打包，同时还能压缩。因此用 `tar` 的情况更多一些。

## 4.7 网络命令

### `write`

`write [用户名]` 给指定的用户发消息，只要用户在线，就能收到，不在线则收不到。

信息可以是多行的，可以用 Enter 换行，删除需要用 Ctrl+Backspace，发送并退出用 Ctrl+D。

```text
###########################################
julian@ubuntu2204:~/temp$ write karl
hello karl,    
this is julian

###########################################
karl@ubuntu2204:~$ 
Message from julian@ubuntu2204 on pts/0 at 00:56 ...
hello karl,
this is julian
EOF
```

对方收到消息后，回车一下就可以继续编写命令。

### `wall`

`wall [信息]` 给所有在线用户发送消息，包括自己。即 write all

```text
###########################################
julian@ubuntu2204:~/temp$ wall hello everyone

Broadcast message from julian@ubuntu2204 (pts/0) (Sun Nov 26 00:59:01 2023):   

hello everyone

###########################################
karl@ubuntu2204:~$

Broadcast message from julian@ubuntu2204 (pts/0) (Sun Nov 26 00:59:01 2023):   

hello everyone
```

不过这种信息会中断正在进行的命令操作，因此并不推荐。

### `ping`

`-c` 可以设置次数。

### `mail`

Ubuntu 22.04.3 中需要安装 `mailutils` 才能使用，默认没有安装。

`mail [用户名]` 开始撰写新邮件，Cc 是抄送给谁，Subject 填入主题，然后就可以写内容了。跟 `write` 一样，可以多行，Ctrl+D 结束发送。

对方输入 `mail` 可以查看收件箱里有多少邮件，进入后，`help` 可以查看支持的命令，`h` 列举所有邮件，输入邮件前的序号，可以查看邮件内容，`d 1` 删除序号为 1 的邮件。

退出时貌似只有用 `q[uit]` 才能正确保存邮件的查看和删除状态，用 `ex[it]` 不行。

### `last`

查看所有用户以往的全部登录信息。

### `lastlog`

查看每一个用户上一次的登录信息。（包括伪用户）

`-u` 参数加 uid 可以只查看这一个用户的上次登录信息。uid 可以通过 `id` 查看。

```text
karl@ubuntu2204:~$ id karl
uid=1001(karl) gid=1001(karl) groups=1001(karl)

karl@ubuntu2204:~$ lastlog -u 1001
Username         Port     From             Latest
karl             pts/1    192.168.113.15   Sun Nov 26 20:06:32 -0400 2023
```

### `traceroute`

Ubuntu 22.04.3 需要安装 `traceroute` 才能使用，默认没有安装。

可以追踪访问到某个目标地址中间经过了多少路由节点。

> Windows 里叫 `tracert`

### `netstat`

`-t` 查看 TCP 连接，`-u` 查看 UDP 连接，`-l` 查看监听端口，`-n` 不解析为域名，即显示 IP 地址（不加就显示域名）

```text
julian@ubuntu2204:~$ netstat -tuln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
...
```

Rece-Q 和 Send-Q 为 0 表示一切正常。

`-a` 可以查看所有连接

```text
julian@ubuntu2204:~$ netstat -a | less
```

除了显示正在监听的端口外，也能显示已建立连接的端口（ESTABLISHED），还能显示其他程序的联网信息。内容很多，所以加了 `less`。

`-r` 显示路由表。

## 4.8 关机重启命令

`shutdown -h now` 现在立刻关机。

`shutdown -r now` 现在立刻重启。

不加 `now` 默认会等一分钟。

`shutdown` 会在关机前处理好正在运行的服务。

`init --help` 了解系统的 7 个运行级别。

`runlevel` 查看当前的运行级别。

# 第五讲 文本编辑器 Vim

## 5.1 Vim 常用操作

Vim 分为三个工作模式，分别为命令模式、插入模式、编辑模式。命令模式下每敲的一个字母都会被当作命令处理，比如敲 `i a o` 都会进入插入模式，插入模式即可以修改文本，编辑模式是在命令模式下敲 `:` 进入，可以编辑命令，比如保存退出 `:wq`。

### 插入命令

| 命令  | 作用         |
|:--- | ---------- |
| `a` | 在光标所在字符后插入 |
| `A` | 在光标所在行行尾插入 |
| `i` | 在光标所在字符前插入 |
| `I` | 在光标所在行行首插入 |
| `o` | 在光标下插入新行   |
| `O` | 在光标上插入新行   |

### 移动命令

| 命令          | 作用         |
|:----------- | ---------- |
| `:set nu`   | 设置行号       |
| `:set nonu` | 取消行号       |
| `gg`        | 到第一行       |
| `G`         | 到最后一行      |
| `nG`        | 到第 n 行     |
| `:n`        | 到第 n 行（推荐） |
| `$`         | 到行尾        |
| `0`         | 到行首        |

### 删除命令

| 命令        | 作用              |
|:--------- | --------------- |
| `x`       | 删除光标所在处的字符      |
| `nx`      | 删除光标所在处后 n 个字符  |
| `dd`      | 删除光标所在行         |
| `ndd`     | 删除 n 行          |
| `dG`      | 删除光标所在行到文件末尾的内容 |
| `D`       | 删除光标所在处到行尾的内容   |
| `:n1,n2d` | 删除第 n1 行到第 n2 行 |

### 复制粘贴命令

| 命令    | 作用          |
|:----- | ----------- |
| `yy`  | 复制当前行       |
| `nyy` | 复制当前行以下 n 行 |
| `dd`  | 剪切当前行       |
| `ndd` | 剪切当前行以下 n 行 |
| `p`   | 粘贴在当前光标所在行下 |
| `P`   | 粘贴在当前光标所在行上 |

> 提示：其实 `dd` 是剪切命令，当不需要粘贴时，它可以当删除命令用。

### 替换和取消命令

| 命令  | 作用                                |
|:--- | --------------------------------- |
| `r` | 替换光标所在处字符（无需进入插入模式）               |
| `R` | 进入替换模式，类似于 Windows 的插入模式，`Esc` 结束 |
| `u` | 取消上一步操作，即撤销                       |

### 字符串命令

| 命令                  | 作用                    |
|:------------------- | --------------------- |
| `/string`           | 从上往下搜索指定字符串           |
| `?string`           | 从下往上搜索指定字符串           |
| `:set ic`           | 搜索时忽略大小写              |
| `:set noic`         | 搜索时考虑大小写              |
| `n`                 | 搜索指定字符串的下一个出现位置       |
| `N`                 | 搜索指定字符串的上一个出现位置       |
| `:%s/old/new/g`     | 全文替换指定的字符串            |
| `:n1,n2s/old/new/g` | 在第 n1 行到第 n2 行替换指定字符串 |

后两个命令的最后把 `g` 换成 `c` 会在每次替换时要求确认。

### 保存退出命令

| 命令                  | 作用                       |
|:------------------- | ------------------------ |
| `:w`                | 保存修改                     |
| `:w /new/file/path` | 另存为指定文件                  |
| `:wq`               | 保存并退出                    |
| `ZZ`                | 保存并退出的快捷键                |
| `:q!`               | 不保存退出                    |
| `:wq!`              | 强制保存退出（文件所有者和 `root` 可用） |

## 5.2 Vim 使用技巧

### 追加内容

`:r /path/to/file` 可以把指定的文件内容追加到当前文件中。

`:!command` 可以在不退出 Vim 的情况下执行命令。

`:r !command` 可以把命令的执行结果追加到当前文件。如 `:r !date` 可以把当前时间追加到文件中。

### 定义快捷键

例如想要添加一个注释行的快捷键，可以使用 `:map ^P I#<ESC>`，其中 `^P` 的输入方式是 `Ctrl+V, Ctrl+P`，有颜色高亮，并非一个尖角加一个大写的 P，`^P` 代表 `Ctrl+P`，后面就是依次要执行的命令。`I` 是跳到行首并进入插入模式，`#` 是插入一个井号，然后 `<ESC>` 是退出插入模式。这样就完成了行注释。此时在行的任意位置键入 `Ctrl+P` 就会在行首增加一个井号，即注释。

如果取消行注释，可以用 `:map ^B 0x`。其中 `^B` 指快捷键 `Ctrl+B`，`0x` 指跳到行首并删除一个字符。

### 连续行注释

原理是字符串替换。

`:n1,n2s/^/#/g` 可以在第 n1 行到第 n2 行的行首添加一个 `#`，也就是连续注释了 n1 到 n2 行。这里 `^` 代表行首，也就是替换行首为某字符。

`:n1,n2s/^#//g` 可以取消 n1 到 n2 行的行首井号。这里的 `^#` 指的是只位于行首的井号，行中的不管（类似于正则），新字符串不写代表空。

`:n1,n2s/^/\/\//g` 可以给 C 文件加注释。这里斜杠是要转义的。

### 替换

`:ab mymail 123@gmail.com` 表示在文档中敲入 `mymail` 后就会被自动替换为 `123@gmail.com`。（敲入 `mymail` 后需要空格或回车才会生效。）

### 永久生效

以上的所有设置只存储在了内存中，重启后就没了，想要保留设置就需要修改家目录下的 `.vimrc` 文件。

```vim
set nu
map ^P I#<ESC>
map ^B 0x
ab myname Julian
```

保存文件后，每次用 Vim 打开文件都会执行这些命令。注意，文件中 `^P` 的输入方式依然为 `Ctrl+V, Ctrl+P`，并非一个尖角加一个大写的 P。


