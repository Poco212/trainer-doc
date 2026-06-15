## config
```
mkdir -p .config/containers/slims/data .config/containers/slims/www
```
```
cd .config/containers/slims
```
## podman
### podman network
```
podman network create server_net
```
### container database
```
podman run -d \
  --name mariadb \
  --network server_net \
  -p 127.0.0.1:3306:3306 \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=slimsdb \
  -e MYSQL_USER=slimsuser \
  -e MYSQL_PASSWORD=password \
  -v ~/.config/containers/slims/data:/var/lib/mysql \
  docker.io/library/mariadb:latest
```
### container service
```

```
### container app atau public
```
wget https://github.https://github.com/slims/slims9_bulian/releases/download/v9.7.2/slims9_bulian-9.7.2.zip
```
```
unzip slims9_bulian-9.7.2.zip -d www
```
```
cd www/slims9_bulian-9.7.2
```
```
podman build -t slims-php .
```
```
podman run -d \
  --name slims \
  --network server_net \
  -p 8080:80 \
  -v ~/.config/containers/slims/www/slims9_bulian-9.7.2:/var/www/html \
  slims-php
```
