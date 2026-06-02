## preparation
```
sudo pacman -S php php-fpm php-gd mariadb nginx-mainline
```
## service
### nginx
```
sudo systemctl start nginx
```
```
sudo systemctl enable nginx
```
### mariadb
```
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```
```
sudo systemctl start mariadb
```
```
sudo systemctl enable mariadb
```
## database
```
sudo mysql_secure_installation
```
```
sudo mysql -u root -p
```
```
CREATE DATABASE [database];
```
```
CREATE USER '[user]'@'localhost' IDENTIFIED BY '[password]';
```
```
GRANT ALL PRIVILEGES ON [database].* TO '[user]'@'localhost';
```
```
FLUSH PRIVILEGES;
```
```
exit;
```
## config
### slims packge
```
wget https://github.com/slims/slims9_bulian/releases/download/v9.7.2/slims9_bulian-9.7.2.tar.gz
```
```
sudo mkdir -p /var/www/html
```
```
sudo tar -xf slims9_bulian-9.7.2.tar.gz -C /var/www/html/
```
```
sudo mv /var/www/html/slims9_bulian-9.7.2 /var/www/html/slims
```
### nginx
```
sudo mkdir -p /etc/nginx/sites-available
```
```
sudo mkdir -p /etc/nginx/sites-enabled
```
```
sudo nvim /etc/nginx/nginx.conf
```
commenting bagian yang dibawah
```
    #server {
    #    listen       80;
    #    server_name  localhost;

        #access_log  logs/host.access.log  main;

    #    location / {
    #        root   /usr/share/nginx/html;
    #        index  index.html index.htm;
    #    }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
    #     error_page   500 502 503 504  /50x.html;
    #    location = /50x.html {
    #        root   /usr/share/nginx/html;
    #    }

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
	
    #}
```
setelah dicommenting, tambahkan ke paling bawah. sebelum tutup `}` terakhir
```
include /etc/nginx/sites-enabled/*;
```
```
sudo nvim /etc/nginx/sites-available/slims.conf
```
tambahkan sesuai dengan dibawah
```
server {
    listen 80;
    server_name slims.example.org;
    return 301 https://$host$request_uri;
    root /var/www/html/slims;
}
```
```
sudo ln -s /etc/nginx/sites-available/slims.conf /etc/nginx/sites-enabled/
```
```
sudo nginx -t
```
```
sudo systemctl restart nginx
```
## access
akses di browser
```
http://ip_address/slims
```
