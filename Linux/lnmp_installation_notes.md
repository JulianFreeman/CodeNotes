# 安装 LNMP 环境

> 参考 https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-22-04

## 安装 Nginx 1.24

> 参考 https://nginx.org/en/linux_packages.html#Ubuntu

安装依赖

```sh
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
```

导入 nginx 的官方密钥来验证包

```sh
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

校验下载的文件包含正确的密钥

```sh
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```

输出应该包含完整的指纹 `573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62`，如下所示

```text
pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
      573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
uid                      nginx signing key <signing-key@nginx.com>
```

设置 nginx 软件源的 stable 版本

```sh
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

如果要 mainline 版本则用如下命令

```sh
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

设置其优先级高于官方发布的软件源

```sh
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

安装 nginx

```sh
sudo apt update
sudo apt install nginx
```

安装后默认 nginx 不会启动，需要手动启动

```sh
sudo systemctl start nginx
systemctl status nginx
```

## 安装 MariaDB 10.11

> 参考 https://mariadb.org/download/?t=repo-config&d=22.04+%22jammy%22&v=10.11&r_m=osuosl

导入 MariaDB 仓库密钥

```sh
sudo apt install apt-transport-https curl
sudo mkdir -p /etc/apt/keyrings
sudo curl -o /etc/apt/keyrings/mariadb-keyring.pgp 'https://mariadb.org/mariadb_release_signing_key.pgp'
```

在 `/etc/apt/sources.list.d` 下创建一个文件（比如 `/etc/apt/sources.list.d/mariadb.sources`）然后添加如下内容

```text
# MariaDB 10.11 repository list - created 2024-01-06 00:18 UTC
# https://mariadb.org/download/
X-Repolib-Name: MariaDB
Types: deb
# deb.mariadb.org is a dynamic mirror if your preferred mirror goes offline. See https://mariadb.org/mirrorbits/ for details.
# URIs: https://deb.mariadb.org/10.11/ubuntu
URIs: https://ftp.osuosl.org/pub/mariadb/repo/10.11/ubuntu
Suites: jammy
Components: main main/debug
Signed-By: /etc/apt/keyrings/mariadb-keyring.pgp
```

然后就可以安装 MariaDB 了

```sh
sudo apt update
sudo apt install mariadb-server
```

如果你更喜欢传统的一行式风格，可以在 `/etc/apt/sources.list.d/mariadb.list` 文件中添加如下内容

```text
# MariaDB 10.11 repository list - created 2024-01-06 00:18 UTC
# https://mariadb.org/download/
# deb.mariadb.org is a dynamic mirror if your preferred mirror goes offline. See https://mariadb.org/mirrorbits/ for details.
# deb [signed-by=/etc/apt/keyrings/mariadb-keyring.pgp] https://deb.mariadb.org/10.11/ubuntu jammy main
deb [signed-by=/etc/apt/keyrings/mariadb-keyring.pgp] https://ftp.osuosl.org/pub/mariadb/repo/10.11/ubuntu jammy main
# deb-src [signed-by=/etc/apt/keyrings/mariadb-keyring.pgp] https://ftp.osuosl.org/pub/mariadb/repo/10.11/ubuntu jammy main
```

安装后查看服务是否启动

```sh
systemctl status mysqld
```

### 安装后设置

默认 root 用户无密码

```sh
sudo mysql -u root -p
```

查看认证方式

```sql
use mysql;
desc user;
select User, plugin from user;
```

默认都是 `mysql_native_password`。`exit` 退出。

运行

```sh
sudo mysql_secure_installation
```

## 安装 PHP 8.1

```sh
sudo apt install php8.1-fpm php-mysql
```

查看服务是否运行

```sh
systemctl status php8.1-fpm
```

### 安装后配置

```sh
sudo vim /etc/php/8.1/fpm/pool.d/www.conf
```

修改如下（将 `www-data` 换为 `nginx`）

```text
user = nginx
group = nginx

listen.owner = nginx
listen.group = nginx
```

重启服务

```sh
sudo systemctl restart php8.1-fpm
```

修改 nginx 配置文件（`sudo vim /etc/nginx/conf.d/default.conf`）如下

```text
...
    location ~ \.php$ {
        root           /usr/share/nginx/html;
        fastcgi_pass   unix:/run/php/php8.1-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
...
```

重新加载配置文件

```sh
sudo systemctl reload nginx
```

创建 `/usr/share/nginx/html/info.php` 文件，并输入如下内容

```php
<?php
phpinfo();
?>
```

浏览器访问 http://[ip]/info.php，查看是否正常。

正常则删除 `/usr/share/nginx/html/info.php` 文件。

### 测试 PHP 使用 MariaDB

以 root 身份进入数据库

```sh
sudo mysql
```

创建测试数据库

```sql
create database test_db;
```

创建用户（三种方式）

```sql
-- 第一种，默认 mysql_native_password 认证，明文密码
CREATE USER test_user@'%' IDENTIFIED BY '123456';

-- 第二种，默认 mysql_native_password 认证，哈希密码
select password('654321');
-- +-------------------------------------------+
-- | password('654321')                        |
-- +-------------------------------------------+
-- | *2A032F7C5BA932872F0F045E0CF6B53CF702F2C5 |
-- +-------------------------------------------+
create user demo_user@'%' identified by password '*2A032F7C5BA932872F0F045E0CF6B53CF702F2C5';

-- 第三种，指定 mysql_native_password 认证，哈希密码
select password('123456');
-- +-------------------------------------------+
-- | password('123456')                        |
-- +-------------------------------------------+
-- | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
-- +-------------------------------------------+
create user example_user@'%' identified with mysql_native_password using '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

`identified with` 等同于 `identified via`，但是 `by` 并不等同于 `using`。且 `identified with` 后面不能跟 `by` 只能跟 `using`。

赋予用户权限

```sql
grant all on test_db.* to test_user@'%';
```

退出并使用测试用户重新登入

```sh
mysql -u test_user -p
```

查看数据库

```sql
show databases;
-- +--------------------+
-- | Database           |
-- +--------------------+
-- | information_schema |
-- | test_db            |
-- +--------------------+
```

创建表

```sql
use test_db;
create table todo_list (item_id int auto_increment, content varchar(255), primary key(item_id));
```

插入数据

```sql
insert into todo_list (content) values ('My first item'), ('My second item'), ('My third item'), ('My fourth item');

select * from todo_list;
-- +---------+----------------+
-- | item_id | content        |
-- +---------+----------------+
-- |       1 | My first item  |
-- |       2 | My second item |
-- |       3 | My third item  |
-- |       4 | My fourth item |
-- +---------+----------------+
```

退出。

创建 `/usr/share/nginx/html/todo_list.php` 文件，并输入如下内容

```php
<?php
$user = "test_user";
$password = "123456";
$database = "test_db";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>"; 
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
?>
```

浏览器访问 http://[ip]/todo_list.php，如果看到 TODO 清单，则测试通过。




