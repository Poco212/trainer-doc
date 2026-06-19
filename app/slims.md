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
    app01:
        image: slimsofficial/slims:latest
        restart: always
        networks: 
            - slims-net
        container_name: slims-app
        ports:
            - "8080:80"
        #    - "443:443"
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
    server_name domain;

    location / {
        proxy_pass http://ip_server1:port_apk;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
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
