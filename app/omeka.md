# server 1
## prepare
```
sudo pacman -S podman-compose nginx
```
```
mkdir -p .config/containers/omeka .config/containers/omeka/config .config/containers/omeka/files
```
```
cd .config/containers/omeka
```
```
chmod -R 777 files
```
```
chmod -R 777 config
```
```
nvim config/database.ini
```
isi
```
user     = "omeka"
password = "omeka"
dbname   = "omeka"
host     = "db"
port     = "3306"
```
```
nvim docker-compose.yml
```
```
version: '2'
services:
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: omeka
      MYSQL_DATABASE: omeka
      MYSQL_USER: omeka
      MYSQL_PASSWORD: omeka

  omeka-s:
    depends_on:
      - db
    build: ./
    image: elestio/omeka:latest
    ports:
      - "8081:80"
    volumes:
      - ./files:/var/www/html/omeka-s/files
      - ./config/database.ini:/var/www/html/omeka-s/config/database.ini
    restart: always
```
```
podman compose up -d
```
