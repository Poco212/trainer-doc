# server 1 (database)
## prepare
```
sudo pacman -S podman-compose
```
## config
```
mkdir -p .config/containers/database
```
```
cd .config/containers/database
```
```
nvim envi-db.env
```
isi 
```
MYSQL_DATABASE=slims
MYSQL_ROOT_PASSWORD=mypassword
MYSQL_USER=slims_user
MYSQL_PASSWORD=s0beautifulday 
```
## running
```
podman run -d --name slims-db --restart always --network slims-net --env-file db_default.env -p 3306:3306 -v ./dbdata:/var/lib/mysql:Z mysql:5.7 --sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION --max_allowed_packet=1024M
```
