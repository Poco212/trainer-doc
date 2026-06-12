# 1. Install Docker

Update sistem:

```bash
sudo pacman -Syu
```

Install Docker:

```bash
sudo pacman -S docker
```

Aktifkan Docker:

```bash
sudo systemctl enable --now docker
```

Verifikasi:

```bash
docker version
docker info
```

---

# 2. Inisialisasi Docker Swarm

Pada node manager:

```bash
docker swarm init --advertise-addr [ip manager]
```

Verifikasi:

```bash
docker node ls
```

Harus muncul status:

```text
Leader
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

---

# 3. Clone Repository OpenDocMan

```bash
sudo mkdir -p /opt/stacks
cd /opt/stacks

git clone https://github.com/opendocman/opendocman.git

cd opendocman
```

---

# 4. Generate Environment File

Jalankan script bawaan repository:

```bash
./scripts/generate-env-secrets.sh
```

---

# 6. Build Image

Karena Docker Swarm tidak mendukung build saat deploy:

```bash
docker build -t opendocman:patched .
```

Verifikasi:

```bash
docker images | grep opendocman
```

---

# 7. Buat Overlay Network

```bash
docker network create \
  --driver overlay \
  opendocman-net
```

Verifikasi:

```bash
docker network ls
```

---

# 8. Siapkan Direktori Data

```bash
sudo mkdir -p /srv/opendocman/mysql
sudo mkdir -p /srv/opendocman/files
sudo mkdir -p /srv/opendocman/config
```

Permission:

```bash
sudo chmod -R 755 /srv/opendocman
```

---

# 9. Buat stack.yml

```bash
nvim stack.yml
```

Isi:

```yaml
version: "3.9"

services:

  db:
    image: mariadb:10.11

    env_file:
      - .env

    volumes:
      - /srv/opendocman/mysql:/var/lib/mysql

    networks:
      - opendocman-net

    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.role == backend

  opendocman:
    image: opendocman:patched

    env_file:
      - .env

    volumes:
      - /srv/opendocman/files:/var/www/html/files-data
      - /srv/opendocman/config:/var/www/html/docker-configs

    ports:
      - target: 80
        published: 8080
        protocol: tcp
        mode: ingress

    networks:
      - opendocman-net

    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.role == frontend

networks:
  opendocman-net:
    external: true
```

---

# 10. Deploy Stack

```bash
docker stack deploy -c stack.yml opendocman
```

---

# 11. Verifikasi Service

```bash
docker stack services opendocman
```

Harus:

```text
NAME                         REPLICAS
opendocman_db                1/1
opendocman_opendocman        1/1
```

Jika ada masalah:

```bash
docker service logs opendocman_db

docker service logs opendocman_opendocman
```

---

# 12. Install Apache

Install:

```bash
sudo pacman -S apache
```

Aktifkan:

```bash
sudo systemctl enable --now httpd
```

Verifikasi:

```bash
systemctl status httpd
```

---

# 13. Aktifkan Module Reverse Proxy

Edit:

```bash
sudo nvim /etc/httpd/conf/httpd.conf
```

Pastikan tidak dikomentari:

```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule headers_module modules/mod_headers.so
LoadModule rewrite_module modules/mod_rewrite.so
```

---

# 14. Buat Virtual Host OpenDocMan

Buat file:

```bash
sudo nvim /etc/httpd/conf/conf.d/opendocman.conf
```

Isi:

```apache
<VirtualHost *:80>

    ServerName [domain]

    ProxyPreserveHost On
    ProxyRequests Off

    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/

    ErrorLog "/var/log/httpd/opendocman-error.log"
    CustomLog "/var/log/httpd/opendocman-access.log" combined

</VirtualHost>
```

---

# 15. Include Virtual Host

Edit:

```bash
sudo nvim /etc/httpd/conf/httpd.conf
```

Tambahkan di bagian bawah:

```apache
Include conf/extra/opendocman.conf
```

---

# 16. Validasi Konfigurasi Apache

```bash
sudo apachectl configtest
```

Harus:

```text
Syntax OK
```

Restart Apache:

```bash
sudo systemctl restart httpd
```

---

# 17. Firewall

Jika menggunakan firewalld:

```bash
sudo firewall-cmd --permanent --add-service=http

sudo firewall-cmd --permanent --add-service=https

sudo firewall-cmd --reload
```

---

# 18. Tambahkan DNS atau Hosts

Contoh:

```text
192.168.1.100 docs.example.com
```

Atau pada `/etc/hosts`:

```text
192.168.1.100 docs.example.com
```

---

# 19. Akses Installer

Buka:

```text
http://docs.example.com
```

Selesaikan wizard instalasi OpenDocMan.

---

# 20. Monitoring

Status stack:

```bash
docker stack services opendocman
```

Status task:

```bash
docker stack ps opendocman
```

Log database:

```bash
docker service logs opendocman_db
```

Log aplikasi:

```bash
docker service logs opendocman_opendocman
```

Jika nanti Anda ingin menambahkan HTTPS, saya sarankan langsung menggunakan sertifikat Let's Encrypt atau sertifikat internal mkcert dan mengubah Apache menjadi reverse proxy TLS pada port 443, lalu set:

```env
ODM_HOSTNAME=docs.example.com
```

sesuai hostname yang benar-benar digunakan untuk mengakses OpenDocMan.
