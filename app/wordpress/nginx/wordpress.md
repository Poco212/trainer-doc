# Install WordPress Native di Arch Linux Menggunakan Nginx Reverse Proxy

## Step 1

Update sistem:

```bash
sudo pacman -Syu
```

---

## Step 2

Install paket yang dibutuhkan:

```bash
sudo pacman -S nginx mariadb php php-fpm wget
```

---

## Step 3

Inisialisasi MariaDB:

```bash
sudo mariadb-install-db \
  --user=mysql \
  --basedir=/usr \
  --datadir=/var/lib/mysql
```

---

## Step 4

Aktifkan MariaDB:

```bash
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

Verifikasi:

```bash
sudo systemctl status mariadb
```

---

## Step 5

Amankan instalasi MariaDB:

```bash
sudo mysql_secure_installation
```

Rekomendasi:

| Option                                | Value     |
| ------------------------------------- | --------- |
| Switch to unix_socket authentication  | n         |
| Change root password                  | n         |
| Remove anonymous users                | Y         |
| Disallow root login remotely          | Y         |
| Remove test database and access to it | n         |
| Reload privilege tables now           | n         |

---

## Step 6

Masuk ke MariaDB:

```bash
sudo mysql -u root -p
```

Buat database:

```sql
CREATE DATABASE wordpress CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Buat user:

```sql
CREATE USER 'wpuser'@'localhost'
IDENTIFIED BY 'PasswordKuat';
```

Berikan hak akses:

```sql
GRANT ALL PRIVILEGES
ON wordpress.*
TO 'wpuser'@'localhost';
```

Terapkan perubahan:

```sql
FLUSH PRIVILEGES;
```

Keluar:

```sql
EXIT;
```

---

## Step 7

Edit konfigurasi PHP:

```bash
sudo nvim /etc/php/php.ini
```

Aktifkan ekstensi berikut:

```ini
extension=mysqli
extension=gd
extension=mbstring
extension=intl
extension=xml
```

---

## Step 8

Konfigurasi PHP-FPM:

```bash
sudo nvim /etc/php/php-fpm.d/www.conf
```

Pastikan:

```ini
user = http
group = http

listen = /run/php-fpm/php-fpm.sock

listen.owner = http
listen.group = http
listen.mode = 0660
```

---

## Step 9

Aktifkan PHP-FPM:

```bash
sudo systemctl enable php-fpm
sudo systemctl start php-fpm
```

Verifikasi:

```bash
sudo systemctl status php-fpm
```

---

## Step 10

Buat direktori WordPress:

```bash
sudo mkdir -p /srv/http/wordpress
```

Masuk ke direktori sementara:

```bash
cd /tmp
```

Download WordPress:

```bash
wget https://wordpress.org/latest.tar.gz
```

Ekstrak arsip:

```bash
tar -xzf latest.tar.gz
```

Salin file WordPress:

```bash
sudo cp -a wordpress/. /srv/http/wordpress/
```

---

## Step 11

Atur kepemilikan file:

```bash
sudo chown -R http:http /srv/http/wordpress
```

---

## Step 12

Masuk ke direktori WordPress:

```bash
cd /srv/http/wordpress
```

Buat file konfigurasi:

```bash
sudo cp wp-config-sample.php wp-config.php
```

Edit konfigurasi:

```bash
sudo nvim wp-config.php
```

Sesuaikan parameter database:

```php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'PasswordKuat');
define('DB_HOST', 'localhost');
```

---

## Step 14

Edit konfigurasi Nginx:

```bash
sudo nvim /etc/nginx/nginx.conf
```

Gunakan konfigurasi berikut:

```nginx

#user http;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


# Load all installed modules
include modules.d/*.conf;

events {
    worker_connections  1024;
}


http {
    types_hash_max_size 4096;
    types_hash_bucket_size 128;


    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #access_log  logs/host.access.log  main;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }


        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    server {
        listen 127.0.0.1:8080;

        root /srv/http/wordpress;
        index index.php;

        location / {
            try_files $uri $uri/ /index.php?$args;
        }

        location ~ \.php$ {
            include fastcgi.conf;
            fastcgi_pass unix:/run/php-fpm/php-fpm.sock;
        }
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

---

## Step 15

Validasi konfigurasi Nginx:

```bash
sudo nginx -t
```

Output yang diharapkan:

```text
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

## Step 16

Aktifkan Nginx:

```bash
sudo systemctl enable nginx
sudo systemctl restart nginx
```

Verifikasi:

```bash
sudo systemctl status nginx
```

---

## Step 18

Akses WordPress:

```text
http://localhost
```

atau

```text
http://IP-SERVER
```

--- 

## firewalld

```
sudo firewall-cmd --list-all-zone
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --reload
```

---

## Step 19

Lengkapi wizard instalasi WordPress:

* Site Title
* Username Administrator
* Password Administrator
* Email Administrator

Klik **Install WordPress**.

---

## Step 20

Login ke dashboard WordPress:

```text
http://IP-SERVER/wp-admin
```

atau

```text
http://localhost/wp-admin
```

---

## Troubleshooting

### Nginx gagal start

Periksa konfigurasi:

```bash
sudo nginx -t
```

Periksa log:

```bash
sudo journalctl -xeu nginx.service
```

---

### No space left on device

Periksa kapasitas disk:

```bash
df -h
```

Periksa inode:

```bash
df -i
```

---

### PHP-FPM tidak berjalan

Periksa status:

```bash
sudo systemctl status php-fpm
```

Periksa log:

```bash
sudo journalctl -xeu php-fpm.service
```

---

### WordPress tidak dapat terhubung ke database

Verifikasi kredensial pada:

```text
/srv/http/wordpress/wp-config.php
```

Lakukan pengujian login MariaDB:

```bash
mysql -u wpuser -p wordpress
```
