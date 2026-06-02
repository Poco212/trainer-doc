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
tambahkan ke paling bawah. sebelum tutup `}` terakhir
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
