Berikut langkah lengkap dari awal untuk membuat cluster **Docker Swarm 2 node**:

```text
PC1 (Manager)
192.168.1.10
├── Nginx / Traefik
└── Docker Swarm Manager

PC2 (Worker)
192.168.1.20
├── ArchivesSpace
├── MariaDB
└── Solr
```

Asumsi:

* Arch Linux di kedua PC
* Docker sudah terinstall
* Kedua PC bisa saling ping
* Firewall membuka port swarm

## 1. Install Docker

Di kedua node:

```bash
sudo pacman -S docker
```

Aktifkan service:

```bash
sudo systemctl enable --now docker
```

Cek:

```bash
docker version
```

---

## 2. Buka Port Swarm

Pastikan port berikut bisa diakses antar node:

```text
2377/tcp   Swarm management
7946/tcp   Node communication
7946/udp   Node communication
4789/udp   Overlay network
```

Jika menggunakan firewalld:

```bash
sudo firewall-cmd --permanent --add-port=2377/tcp
sudo firewall-cmd --permanent --add-port=7946/tcp
sudo firewall-cmd --permanent --add-port=7946/udp
sudo firewall-cmd --permanent --add-port=4789/udp
sudo firewall-cmd --reload
```

---

## 3. Inisialisasi Swarm di PC1

Di PC1:

```bash
docker swarm init --advertise-addr 192.168.1.10
```

Output:

```text
docker swarm join \
--token SWMTKN-xxxx \
192.168.1.10:2377
```

Simpan token tersebut.

---

## 4. Join PC2 ke Cluster

Di PC2:

```bash
docker swarm join \
--token SWMTKN-xxxx \
192.168.1.10:2377
```

---

## 5. Verifikasi Cluster

Di PC1:

```bash
docker node ls
```

Contoh:

```text
ID          HOSTNAME   STATUS
abc123      pc1        Ready
xyz789      pc2        Ready
```

---

## 6. Beri Label Node

Supaya service bisa ditempatkan di node tertentu.

Di PC1:

```bash
docker node update \
  --label-add role=frontend \
  pc1
```

```bash
docker node update \
  --label-add role=backend \
  pc2
```

Cek:

```bash
docker node inspect pc1 --pretty
docker node inspect pc2 --pretty
```

---

## 7. Buat Overlay Network

Di PC1:

```bash
docker network create \
  -d overlay \
  --attachable \
  archivesspace-net
```

Cek:

```bash
docker network ls
```

---

## 8. Buat Direktori Stack

Di PC1:

```bash
mkdir ~/swarm
cd ~/swarm
```

---

## 9. Buat stack.yml

```yaml
version: "3.9"

services:

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    networks:
      - archivesspace-net
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.role == frontend

  mariadb:
    image: mariadb:11
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: archivesspace
      MYSQL_USER: archivesspace
      MYSQL_PASSWORD: archivesspace
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      - archivesspace-net
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.role == backend

  solr:
    image: archivesspace/solr:latest
    networks:
      - archivesspace-net
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.role == backend

  archivesspace:
    image: archivesspace/archivesspace:latest
    networks:
      - archivesspace-net
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.role == backend

networks:
  archivesspace-net:
    external: true

volumes:
  mariadb_data:
```

Simpan sebagai:

```bash
nano stack.yml
```

---

## 10. Deploy Stack

Di manager (PC1):

```bash
docker stack deploy \
  -c stack.yml \
  archivesspace
```

---

## 11. Lihat Service

```bash
docker stack services archivesspace
```

atau:

```bash
docker service ls
```

---

## 12. Lihat Penempatan Service

```bash
docker service ps archivesspace_nginx
```

```bash
docker service ps archivesspace_archivesspace
```

Contoh:

```text
archivesspace_nginx          Running   pc1
archivesspace_mariadb        Running   pc2
archivesspace_solr           Running   pc2
archivesspace_archivesspace  Running   pc2
```

---

## 13. Update Stack

Jika mengubah `stack.yml`:

```bash
docker stack deploy \
  -c stack.yml \
  archivesspace
```

Swarm akan melakukan rolling update otomatis.

---

## 14. Hapus Stack

```bash
docker stack rm archivesspace
```

---

## 15. Monitoring

Lihat node:

```bash
docker node ls
```

Lihat service:

```bash
docker service ls
```

Lihat task:

```bash
docker service ps archivesspace_archivesspace
```

Lihat log:

```bash
docker service logs -f archivesspace_archivesspace
```

### Catatan Penting untuk ArchivesSpace

Image `archivesspace/archivesspace` tidak langsung siap jalan hanya dengan menjalankan container. ArchivesSpace memerlukan konfigurasi database, Solr, dan file konfigurasi aplikasi. Untuk deployment yang benar-benar berfungsi, Anda perlu menyesuaikan environment variable dan konfigurasi sesuai dokumentasi resmi ArchivesSpace.

Saya sarankan langkah berikutnya adalah membuat stack yang mengikuti contoh deployment resmi ArchivesSpace (MariaDB + Solr + ArchivesSpace + Nginx/Traefik) agar aplikasi bisa langsung digunakan setelah `docker stack deploy`.
