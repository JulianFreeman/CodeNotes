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

`ln -s` 可以创建软链接，不过指定源路径时必须用 **绝对路径**，不能用相对路径。使用相对路径建立的软链接用 `ls` 看是红色的，也无法读取，可能意味着无效。用绝对路径创建的软链接显示蓝色，可以读取，是有效的。

`ln` 创建硬链接则没有这个要求，相对路径也可以。

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



## 4.3 文件搜索命令

## 4.4 帮助命令

## 4.5 用户管理命令

## 4.6 压缩解压命令

## 4.7 网络命令

## 4.8 关机重启命令
