# install wordpress manual arch

## step 1

- install apache

```
sudo pacman -S apache
```

## step 2

- start apache service

```
sudo systemctl start httpd
```

## step 3

- enable apache service

```
sudo systemctl enable httpd
```

## step 4

- install mariadb, php, php fpm

```
sudo pacman -S mariadb php php-fpm
```

## step 5

- install database pada mariadb

```
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

## step 6

- enable mariadb service

```
sudo systemctl enable mariadb
```

## step 7

- start mariadb service

```
sudo systemctl start mariadb
```

## step 8

- membuat password untuk user root mariadb

```
sudo mysql_secure_installation
```
| Option                                 | [Y/n] |
| -------------------------------------- | ----- |
| Switch to unix_socket authentication   | n     |
| Change the root password?              | n     |
| Remove anonymous users?                | Y     |
| Disallow root login remotely?          | Y     |
| Remove test database and access to it? | n     |
| Reload privilege tables now?           | n     |

## step 9

- masuk ke dalam mariadb menggunakan user root

```
sudo mysql -u root -p
```

note: masukkan password yang telah dibuat tadi

## step 10

- buat database untuk wordpress

```
CREATE DATABASE wordpress;
```

- buat user untuk database wordpress

```
CREATE USER '[user]'@'localhost' IDENTIFIED BY '[password]';
```
note: nama user dan password disesuaikan

- berikan akses ke user

```
GRANT ALL PRIVILEGES ON wordpress.* TO '[user]'@'localhost';
```
note: user disesuaikan

- jalankan command flush privileges

```
FLUSH PRIVILEGES;
```

- keluar dari mariadb

```
exit;
```

## step 11

- masuk ke dalam folder /srv/http

```
cd /srv/http/
```

- download wordpress menggunakan wget

```
sudo wget https://wordpress.org/latest.tar.gz
```
note: jika belum ada wget maka install terlebih dahulu

- unzip file wordpress

```
sudo tar -xvzf latest.tar.gz
```

- pindah kan semua file ke folder /srv/http

```
sudo mv wordpress/* .
```

- hapus folder wordpress dan file zip

```
sudo rm -rf wordpress latest.tar.gz
```

## step 12

- tetapkan izin untuk direktori wordpress

```
sudo chown -R http:http /srv/http/
```

## step 13

- buat file konfigurasi dari sampel yang sudah ada


```
sudo cp /srv/http/wp-config-sample.php /srv/http/wp-config.php
```

## step 14

- edit file konfigurasi temukan seperti dibawah

```php
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', '[user]' );

/** MySQL database password */
define( 'DB_PASSWORD', '[password]' );
```

## step 15

- konfigurasi file utama apache

```
nvim /etc/httpd/conf/httpd.conf
```

- tambahkan pada Loadmodule

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
```

- hilangkan tanda # pada module rewrite menjadi seperti dibawah

```
LoadModule rewrite_module modules/mod_rewrite.so
```

- tambahkan pada baris paling terakhir

```
<FilesMatch "\.php$">
    SetHandler "proxy:unix:/run/php-fpm/php-fpm.sock|fcgi://localhost/"
</FilesMatch>
```

- ubah aksess override pada direktori /srv/http menjadi all

```
AllowOverride All
```

- tambahkan pada module direktori 

```
<IfModule dir_module>
    DirectoryIndex index.php index.html index.htm
</IfModule>
```

## step 16 

- konfigurasi file php.ini

```
sudo nvim /etc/php/php.ini
```

- aktifkan ekstensi mysqli, dengan menghapus tanda #

```
extension=mysqli
extension=gd
```

- tambahkan ekstensi xml dan mbstring

```
extension=xml
extension=mbstring
```

## step 17

- enable php fpm service

```
sudo systemctl enable php-fpm
```

- start php fpm service

```
sudo systemctl start php-fpm
```

## step 18 

- restart apache service

```
sudo systemctl restart httpd
```

## step 19

- akses wordpress melalui web browser

https://localhost

## step 20

- anda diminta untuk membuat user admin untuk pengelolaan wordpress

note: lengkapi sesuai yang anda inginkan

**selamat anda telah berhasil menginstall wordpress secara manual**
