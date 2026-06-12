# docker swarm jellyfin

```
docker swarm init
```

## create jellyfin directory

```
mkdir -p jellyfin/media
```

```
mkdir -p jellyfin/config
```

```
mkdir -p jellyfin/cache
```

## create jellyfin stack config

```
nvim jellyfin/jellyfin.yml
```

```
version: "3.9"

services:
  jellyfin:
    image: jellyfin/jellyfin
    ports:
      - target: 8096
        published: 8096
        protocol: tcp
      - target: 7359
        published: 7359
        protocol: udp
    volumes:
      - /home/kamil/jellyfish/config:/config
      - /home/kamil/jellyfish/cache:/cache
      - /home/kamil/jellyfish/media:/media
    deploy:
      replicas: 3
      placement:
        max_replicas_per_node: 1
      restart_policy:
        condition: any

volumes:
  jellyfin-config:
  jellyfin-cache:
```

```
docker stack deploy -c jellyfin.yml media
```

## list all nodes

```
docker node ls
```

## list all service
```
docker service ls
```
