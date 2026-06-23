# server 1
## prepare
```
sudo pacman -S podman-compose fuse-overlayfs
```
```
mkdir container
```
```
cd container
```
```
nvim storage.conf
```
isi
```

```
```
cd atom
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
podman compose -f docker/docker-compose.dev.yml up -d
```
