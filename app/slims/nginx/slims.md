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
sudo mysql -u root -p
```
```
sudo mysql_secure_installation
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
```
wget https://github.com/slims/slims9_bulian/releases/download/v9.7.2/slims9_bulian-9.7.2.tar.gz
```
```
sudo tar -xf slims9_bulian-9.7.2.tar.gz -C /var/www/html
```
```
sudo mv /var/www/html/slims9_bulian-9.7.2 /var/www/html/slims
```
