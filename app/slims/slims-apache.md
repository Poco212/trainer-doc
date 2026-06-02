# install slims manual arch

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

- install mariadb, php, php fpm, dan php gd

```
sudo pacman -S mariadb php php-fpm php-gd
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

- buat database untuk slims

```
CREATE DATABASE slims;
```

- buat user untuk database slims

```
CREATE USER '[user]'@'localhost' IDENTIFIED BY '[password]';
```
note: nama user dan password disesuaikan

- berikan akses ke user

```
GRANT ALL PRIVILEGES ON slims.* TO '[user]'@'localhost';
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

- Install slims directory dari github

```
wget https://github.com/slims/slims9_bulian/releases/download/v9.7.2/slims9_bulian-9.7.2.tar.gz
```

- pindahkan unzip file slims dan pindahkan ke /srv/http

```
sudo tar -xf slims9_bulian-9.7.2.tar.gz -C /srv/http
```

## step 12
- masuk ke dalam folder /srv/http

```
cd /srv/http/
```

- ganti nama file menjadi slims agar lebih mudah nantinya untuk mengakses

```
sudo mv /srv/http/slims9_bulian-9.7.2 /srv/http/slims
```

## step 13

- tetapkan izin untuk direktori slims

```
sudo chown -R http:http /srv/http/slims
```

## step 14

- konfigurasi file utama apache

```
sudo nvim /etc/httpd/conf/httpd.conf
```

- aktifkan module proxy dan proxy fcgi

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

> setelah itu save

## step 15

- konfigurasi file php.ini

```
sudo nvim /etc/php/php.ini
```

- aktifkan ekstensi mysqli

```
extension=mysqli
extension=gettext
extension=pdo-mysql
extension=gd
```

- tambahkan ekstensi xml dan mbstring

```
extension=xml
extension=mbstring
```

> setelah itu save

## step 16

- enable php fpm service

```
sudo systemctl enable php-fpm
```

- start php fpm service

```
sudo systemctl start php-fpm
```

## step 17

- restart apache service

```
sudo systemctl restart httpd
```

## step 18

- akses slims melalui web browser

http://localhost/slims

## step 19

- klik get started

- klik next

- klik install slims

- diminta untuk mengisikan database informasi, ikuti seperti dibawah

| Database information | Answer     |
| -------------------- | ------     |
| Database host        | localhost  |
| Database port        | 3306       |
| Database name        | slims      |
| Database username    | [user]     |
| Database password    | [password] |

note: user dan password yang dibuat saat membuat user untuk database sebelumnya

## step 20

- membuat super user atau administrator user, disesuaikan dengan keinginan

- setelah itu klik next

## step 21

- klik go to my slims
