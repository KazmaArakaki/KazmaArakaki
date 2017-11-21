# Install Nginx

``` sh
$ sudo vim /etc/yum.repos.d/nginx.repo
```

**nginx.repo**

```
[nginx]
name=Nginx Repo
baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/
gpgkey=http://nginx.org/keys/nginx_signing.key
gpgcheck=1
enabled=1

[nginx-source]
name=Nginx Source
baseurl=http://nginx.org/packages/mainline/centos/7/SRPMS/
gpgkey=http://nginx.org/keys/nginx_signing.key
gpgcheck=1
enebled=0
```

``` sh
$ sudo yum install nginx
```

# Install PHP, PHP-FPM and extensions

``` sh
$ sudo yum install epel-release
$ sudo rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
$ sudo yum install --enablerepo=remi-php71 php php-fpm php-intl php-mbstring php-mysql php-simplexml
```

# Install MySQL (MariaDB)

``` sh
$ sudo vim /etc/yum.repos.d/mariadb.repo
```

**mariadb.repo**

```
[mariadb]
name=MariaDB
baseurl=http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

``` sh
$ sudo yum install mariadb MariaDB-server
```

# Create project directory

``` sh
$ sudo mkdir -p /var/www/cakephp_app
```

# Get composer

``` sh
$ cd /var/www/cakephp_app
$ sudo wget https://getcomposer.org/composer.phar
$ sudo mv composer.phar /usr/local/bin/composer
```

# Install CakePHP

``` sh
$ sudo composer create-project --prefer-dist cakephp/app .
```

# Configure Nginx

``` sh
$ sudo vim /etc/nginx/conf.d/default.conf
```

**default.conf**

```
server {
  listen 80 default_server;
  server_name localhost_http;
  
  root /var/www/cakephp_app/webroot;
  index index.php index.html;

  location / {
    try_files $uri $uri?$args $uri/ /index.php?$uri&$args /index.php?$args;
  }

  location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
    include fastcgi_params;
  }
}
```

``` sh
$ sudo systemctl start nginx
```

# Configure PHP

``` sh
$ sudo vim /etc/php.ini
```

**php.ini** (excerption)

```
;;;;;;;;;;;;;;;;;;;;;;
; Dynamic Extensions ;
;;;;;;;;;;;;;;;;;;;;;;

; If you wish to have an extension loaded automatically, use the following
; syntax:
;
;   extension=modulename.extension
;
; For example, on Windows:
;
;   extension=msql.dll
;
; ... or under UNIX:
;
;   extension=msql.so
;
; ... or with a path:
;
;   extension=/path/to/extension/msql.so
;
; If you only provide the name of the extension, PHP will look for it in its
; default extension directory.
extension=intl.so
```

# Configure PHP-FPM

``` sh
$ sudo vim /etc/php-fpm.d/www.conf
```

**www.conf** (excerption)

```
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
; RPM: apache Choosed to be able to access some dir as httpd
user = nginx
; RPM: Keep a group allowed to write in log dir.
group = nginx
```

``` sh
$ sudo systemctl start php-fpm
```

# Configure MySQL (MariaDB)

``` sh
sudo vim /etc/my.cnf.d/server.cnf
```

**server.cnf** (excerption)

```
[mysqld]
character-set-server=utf8
```

``` sh
$ sudo systemctl start mariadb
$ mysql_secure_installation
```

# Configure CakePHP

``` sh
$ sudo composer install
```
