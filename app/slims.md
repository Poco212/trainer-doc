## config
```
mkdir -p .config/containers/slims/data
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
podman run -d --name mariadb --network server_net -p "127.0.0.1:3306:3306" -e MYSQL_DATABASE=slimsdb MYSQL_USER=slimsuser MYSQL_PASSWORD=password -v ~/.config/containers/slims/data:/var/lib/mysql  docker.io/library/mariadb:latest
```
### container service
```

```
### container app atau public
```
podman run -d --name slims --network server_net
```