# server 1
## prepare
```
sudo pacman -S podman-compose 
```
```
mkdir -p .config/containers/atom
```
```
cd .config/containers/atom
```
```
git clone -b qa/2.x https://github.com/artefactual/atom.git atom
```
```
cd atom
```
```
export COMPOSE_FILE="$PWD/docker/docker-compose.dev.yml"
```
```
sudo podman compose -f docker/docker-compose.dev.yml up -d
```
