# server 1
## prepare
```
sudo pacman -S podman-compose nginx
```
```
mkdir -p .config/containers/omeka
```
```
cd .config/containers/omeka
```
```
nvim database.ini
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
    image: klokantech/omeka-s
    ports:
      - "8081:80"
    volumes:
      - ./modules/:/var/www/html/modules/
      - ./themes/custom/:/var/www/html/themes/custom/
    restart: always
```
```
podman compose up -d
```
