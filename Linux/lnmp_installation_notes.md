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

# 创建证书

WordPress 需要证书。我没有试过没有证书的话会怎样，但不管怎么说，先创建一个吧。

## 无域名自签名证书

自签名证书，顾名思义，就是自己给自己签名的证书，不能被浏览器信任，不过也算是个证书了。

> 以下内容暂时不理解啥意思，纯粹照葫芦画瓢。葫芦地址：https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-22-04

创建 TLS 证书

```sh
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
```

这一步会创建密钥和证书，同时需要提供一些用于记录在证书上的信息，其中最重要的是 `Common Name`，也就是域名，这里就是服务器 IP 地址。

创建这个不知道是什么的 DH 组，这一步会十分花时间

```sh
sudo openssl dhparam -out /etc/nginx/dhparam.pem 4096
```

接下来创建配置文件让 nginx 使用前面创建的证书。

首先创建一个配置样板，指向前面的 SSL 密钥和证书（此处的文件名并非强制要求）（如果没有 `snippets` 目录要先创建）

```sh
sudo vim /etc/nginx/snippets/self-signed.conf
```

填入如下内容

```text
ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
```

然后创建另一个配置样板，并填入一些 SSL 配置

```sh
sudo vim /etc/nginx/snippets/ssl-params.conf
```

```text
ssl_protocols TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_dhparam /etc/nginx/dhparam.pem; 
ssl_ciphers EECDH+AESGCM:EDH+AESGCM;
ssl_ecdh_curve secp384r1;
ssl_session_timeout  10m;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
# Disable strict transport security for now. You can uncomment the following
# line if you understand the implications.
#add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```

下面配置 nginx 以使用 SSL。

首先打开网站的配置文件（比如 `/etc/nginx/conf.d/default.conf`），找到原先的 `server` 块，把 `listen` 改掉，同时加入前面创建的配置文件

```text
server {
    listen 443 ssl;
    include /etc/nginx/snippets/self-signed.conf;
    include /etc/nginx/snippets/ssl-params.conf;

...
}
```

这样当浏览器以 https 协议访问服务器 IP 的时候，就会由这个 `server` 块响应（SSL 是 443 端口处理）。

重新加载 nginx

```sh
sudo systemctl reload nginx
```

此时去浏览器访问 https://[ip]，可能会弹出不认识此证书的警告，但是没关系，接受之后就可以看到网站连接是安全的。

但是此时访问 http://[ip] 是没有响应的，因为没有 `server` 块响应 80 端口的访问。如果我们想把 http 的访问都自动转成 https 的访问，可以再添加一个 `server` 块，填入如下内容

```text
server {
    listen 80;
    server_name [ip];

    return 302 https://$server_name$request_uri;
}
```

其中 [ip] 的部分要填入自己服务器的 IP 地址，如果是有域名的话就填域名，总之这个就是下面的 `return` 里的 `$server_name` 要用到的值。

这个块就是把 http 的请求都跳转为 https 的请求了。

此时浏览器访问 http://[ip] 应该就会看到自动跳转为 https://[ip] 了。

# 安装 Wordpress

> 参考 https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-lemp-on-ubuntu-22-04

## 为 Wordpress 创建数据库

进入数据库

```sh
sudo mysql
```

创建一个新的数据库，这个名字无所谓，不过这里就用简单的 `wordpress` 了

```sql
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

为 Wordpress 创建一个新的用户来操作这个数据库，

```sql
-- 用户名无所谓，这里用 wordpressuser
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY '123456';
-- 授权
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost';
```

退出数据库。

## 安装额外的 PHP 扩展

据说这些是 Wordpress 要用的

```sh
sudo apt install php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip
```

重启服务

```sh
sudo systemctl restart php8.1-fpm
```

## 配置 nginx

前面我们在 `/etc/nginx/conf.d/default.conf` 中添加了自签名证书，这次我们还把这个文件当作 wordpress 的主配置文件。

在响应 443 的 `server` 中添加如下内容

```text
server {
...
    location = /favicon.ico { log_not_found off; access_log off; }
    location = /robots.txt { log_not_found off; access_log off; allow all; }
    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }
...
}
```

简单的 `location`，设置了一些文件不记录日志。因为这些 `location` 里都没有 `root` 和 `index`，因此要在 `server` 内声明 `root` 和 `index`，防止找不到 css 和 js 文件。

然后在 `location /` 中添加这一行

```text
server {
...
    location / {
        ...
        try_files $uri $uri/ /index.php$is_args$args;
    }
...
}
```

重新加载服务

```sh
sudo systemctl reload nginx
```

## 下载 Wordpress

运行如下命令

```sh
cd /tmp
curl -LO https://wordpress.org/latest.tar.gz
tar zxf latest.tar.gz
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
sudo cp -a /tmp/wordpress/. /usr/share/nginx/html/
sudo chown -R nginx:nginx /usr/share/nginx/html
```

这里我们还是用默认的 `/etc/share/nginx/html` 目录作为主目录，然后修改一下它的权限以便 nginx 可以读取并修改里面的文件。

## 设置 Wordpress 配置文件

首先需要一些密钥，执行如下命令获取一些密钥

```sh
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

然后打开 `/usr/share/nginx/html/wp-config.php` 文件，找到类似

```text
...
define('AUTH_KEY',         'put your unique phrase here');
define('SECURE_AUTH_KEY',  'put your unique phrase here');
define('LOGGED_IN_KEY',    'put your unique phrase here');
define('NONCE_KEY',        'put your unique phrase here');
define('AUTH_SALT',        'put your unique phrase here');
define('SECURE_AUTH_SALT', 'put your unique phrase here');
define('LOGGED_IN_SALT',   'put your unique phrase here');
define('NONCE_SALT',       'put your unique phrase here');
...
```

的位置，然后把新生成的密钥替换进去。

下面在同一个文件修改一下数据库的设置（文件开头）

```text
define( 'DB_NAME', 'wordpress' );

define( 'DB_USER', 'wordpressuser' );

define( 'DB_PASSWORD', '123456' );

...
define('FS_METHOD', 'direct');
```

最后一行是写入文件系统的方式，设置为直接写入。

## 在浏览器里进行最后的配置

浏览器访问 https://[ip]/wordpress，看到正常的注册页面，就算成功了。
