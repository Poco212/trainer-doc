```
sudo pacman -S apache2
```
```
sudo systemctl enable --now httpd
```
```
sudo a2enmod proxy
```
```
sudo a2enmod proxy_http
```
```
sudo systemctl restart httpd
```
```
sudo nvim /etc/httpd/conf/httpd.conf
```
uncommenting
```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```
tambahin paling bawah juga
```
Include conf/extra/httpd-vhosts.conf
```
```
sudo nvim /etc/httpd/conf/extra/httpd-vhosts.conf
```
```
<VirtualHost *:80>
    ServerName openkm.domainanda.local

    ProxyRequests Off
    ProxyPreserveHost On

    ProxyPass / http://127.0.0.1:8080
    ProxyPassReverse / http://127.0.0.1:8080
    ErrorLog "/var/log/httpd/openkm-error_log"
    CustomLog "/var/log/httpd/openkm-access_log" common
</VirtualHost>

```
```
sudo apachectl configtest
```
```
sudo systemctl restart httpd
```
```
sudo nvim /etc/hosts
```
```
[masukin ip nya]      [masukin domain sesuai dengan apache]
```

```
sudo systemctl restart httpd
```
