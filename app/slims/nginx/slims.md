## preparation
```
sudo pacman -S php php-fpm php-gd mariadb nginx 
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
