# server 1
## prepare
```
sudo pacman -S podman-compose
```
```
mkdir -p .config/containers
```
```
cd .config/containers
```
```
wget -c https://github.com/slims/docker-compose-for-slims/archive/master.zip
```
```
unzip master.zip 
```
```
mv docker-compose-for-slims-master compose
```
```
cd compose
```
## config
```
sudo nvim /etc/sysctl.d/99-custom.conf
```
isi
```
net.ipv4.ip_unprivileged_port_start=80
```
```
sudo sysctl --system
```
```
mv docker-compose.yaml docker-compose.yaml.bck
```
```
mv docker-compose-redis.yaml docker-compose.yaml
```
```
nvim docker-compose.yaml
```
>[NOTE] pastikan valuenya sama dengan di bawah

```
version: "3.7"
services: 
    db:
        image: mysql:5.7
        restart: always
        networks: 
            - slims-net
        container_name: slims-db
        env_file: 
            - db_default.env
        command: --sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION --max_allowed_packet=1024M
        volumes:
            - "./dbdata:/var/lib/mysql"
        ports:
            - "127.0.0.1:3306:3306"
    redis:
        image: redis:latest
        restart: always
        networks: 
            - slims-net
        container_name: redis
        ports:
            - "127.0.0.1:6379:6379"
    app01:
        image: slimsofficial/slims:latest
        restart: always
        networks: 
            - slims-net
        container_name: slims-app
        ports:
            - "80:80"
            - "443:443"
        volumes:
            - "./app:/var/www/html"
            - "./conf/php/php.ini:/usr/local/etc/php/conf.d/php.ini"
networks: 
    slims-net:
        name: slims-net
        ipam:
            driver: default
```
```
sudo chmod -R 777 app/slims
```
## running
```
podman compose up -d
```

# server 2
```
sudo pacman -S nginx
```
```
sudo mkdir -p /etc/nginx/sites-available
```
```
sudo mkdir -p /etc/nginx/sites-enabled
```
```
sudo nvim /etc/nginx/nginx.conf
```
tambahkan di paling atas
```
user http;
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
    server_name _; # Menggunakan '_' berarti menerima akses dari IP server langsung

    root /var/www/html/slims;
    index index.php index.html index.htm;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Teruskan file PHP ke PHP-FPM socket bawaan Arch Linux
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

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
## hosts
```
sudo nvim /etc/hosts
```
tambahkan
```
ip_address_server2    domain_anda
```
## access
akses di browser
```
http://domain
```
## cek browser
```
http://ip:8080
```
