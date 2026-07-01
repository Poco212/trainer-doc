# Dokumentasi Implemetasi K3s Multi-Node Cluster & Deployment SLiMS
Dokumentasi ini merangkum langkah-langkah setup infrastruktur Kubernetes lokal menggunakan **K3s (Rootful/Standar)** di atas sistem operasi **Arch Linux (Kernel Linux-LTS)** dengan konfigurasi dua komputer fisik (**PC Master** dan **PC Worker**), serta deployment aplikasi **SLiMS (Senayan Library Management System)**.

---

## Arsitektur Jaringan Kluster
*   **PC Master (Control-Plane)**
    *   Nama Node: `backlink`
    *   IP Lokal: `192.168.1.11`
*   **PC Worker (Agent)**
    *   Nama Node: `archlinux`
    *   IP Lokal: (Disesuaikan dengan interface LAN/Wi-Fi Worker)

---

## Bagian 1: Persiapan Sistem (Dijalankan di Kedua PC)

Sebelum instalasi K3s, modul jaringan kernel Linux-LTS harus diaktifkan dan batasan port non-root harus dibuka agar lalu lintas antar-komputer berjalan lancar.

1. **Aktifkan Modul Jaringan Kernel:**
   ```bash
   sudo tee /etc/modules-load.d/k3s.conf > /dev/null <<EOF
   br_netfilter
   overlay
   ip_tables
   EOF

   # Muat modul ke kernel saat ini
   sudo modprobe -a br_netfilter overlay ip_tables
   ```

2. **Konfigurasi Routing Jaringan & Akses Port:**
   Buka izin pencabutan batasan port di bawah 1024 untuk Kubernetes dan aktifkan IP Forwarding.
   ```bash
   sudo tee /etc/sysctl.d/99-k3s-routing.conf > /dev/null <<EOF
   net.ipv4.ip_forward = 1
   net.ipv4.ip_unprivileged_port_start = 0
   EOF

   # Terapkan perubahan sysctl secara instan
   sudo sysctl --system
   ```

---

## Bagian 2: Instalasi K3s Master (Di PC Master - `backlink`)

1. **Eksekusi Skrip Instalasi K3s Standar:**
   ```bash
   curl -sfL https://k3s.io | sh -
   ```

2. **Konfigurasi Hak Akses `kubectl`:**
   Arahkan variabel lingkungan terminal ke berkas konfigurasi kluster baru dan berikan hak akses baca agar perintah `kubectl` dapat dipanggil tanpa `sudo`.
   ```bash
   sudo chmod 644 /etc/rancher/k3s/k3s.yaml
   export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
   
   # Buat konfigurasi permanen di terminal
   echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> ~/.bashrc
   source ~/.bashrc
   ```

3. **Ambil Token Otentikasi Kluster:**
   Token ini diperlukan agar PC Worker dapat bergabung secara aman.
   ```bash
   sudo cat /var/lib/rancher/k3s/server/node-token
   ```
   *(Salin string token panjang yang muncul di layar untuk digunakan di Bagian 3)*.

---

## Bagian 3: Instalasi K3s Worker (Di PC Kedua - `archlinux`)

Gunakan metode **Argumen Perintah** resmi untuk memaksa instalasi murni sebagai agen/worker, guna menghindari sistem membuat master ganda secara lokal.

1. **Hubungkan Agen ke Master:**
   Jalankan perintah ini menggunakan hak akses `sudo` (Ganti token dengan token asli dari Bagian 2):
   ```bash
   curl -sfL https://k3s.io | sudo sh -s - agent \
     --server https://192.168.1.11:6443 \
     --token K100cc99bf87924bd16a16e225215689a790d7e92048157d90ebff2e72694c5327b::server:94c6b0e3b097fa91e2f205fd1d6eda71
   ```

2. **Verifikasi Status Layanan Agen:**
   Pastikan layanan berstatus hijau (*active/running*).
   ```bash
   sudo systemctl status k3s-agent.service
   ```

---

## Bagian 4: Verifikasi Kluster (Di PC Master)

Kembali ke PC Master (`backlink`) untuk memastikan kedua komputer fisik telah terdeteksi dan bersatu membentuk satu kluster produksi:
```bash
kubectl get nodes
```

**Output yang Diharapkan:**
```text
NAME       STATUS   ROLES           AGE     VERSION
backlink   Ready    control-plane   15m     v1.36.2+k3s1
archlinux  Ready    <none>          1m      v1.36.2+k3s1
```

---

## Bagian 5: Deployment Multi-Node Aplikasi SLiMS

Arsitektur ini memisahkan Database ke dalam Pod mandiri dan membagi aplikasi SLiMS Web ke dalam **2 Replika** agar didistribusikan secara otomatis ke PC Master dan PC Worker sekaligus demi efisiensi resource (*Load Balancing*).

1. **Buat File Manifest (`slims-multi-node.yaml`):**
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: slims-db-service
   spec:
     ports:
     - port: 3306
     selector:
       app: slims-db
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: slims-db-pod
     labels:
       app: slims-db
   spec:
     containers:
     - name: mariadb
       image: mariadb:10.6
       env:
       - name: MARIADB_ROOT_PASSWORD
         value: "slimsrootpass"
       - name: MARIADB_DATABASE
         value: "slims_db"
       - name: MARIADB_USER
         value: "slims_user"
       - name: MARIADB_PASSWORD
         value: "slimspassword"
       ports:
       - containerPort: 3306
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: slims-web-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: slims-web
     template:
       metadata:
         labels:
           app: slims-web
       spec:
         containers:
         - name: slims
           image: slimsofficial/slims:latest
           ports:
           - containerPort: 80
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: slims-web-service
   spec:
     type: NodePort
     selector:
       app: slims-web
     ports:
     - port: 80
       targetPort: 80
       nodePort: 30088
   ```

2. **Terapkan Manifest ke Kluster (Dari PC Master):**
   ```bash
   kubectl apply -f slims-multi-node.yaml
   ```

3. **Pantau Distribusi Pod Lintas PC:**
   ```bash
   kubectl get pods -o wide
   ```
   *Perhatikan kolom `NODE`. Anda akan melihat satu pod berjalan di komputer `backlink` dan pod web lainnya berjalan di komputer `archlinux`.*

---

## Bagian 6: Akses & Konfigurasi Aplikasi

Aplikasi SLiMS sekarang dapat dibuka dari perangkat mana pun (Laptop, PC, HP) yang berada di dalam jaringan Wi-Fi/LAN yang sama tanpa perlu melakukan `port-forward`.

1. **Akses Browser:**
   Buka browser dan ketik alamat IP salah satu node diikuti port `30088`. Contoh:
   *   `http://192.168.1.11:30088` (IP Master)

2. **Pengisian Form Database pada Setup Wizard SLiMS:**
   Pada langkah konfigurasi database di web browser, masukkan informasi berikut secara presisi:
   *   **Database Host:** `slims-db-service` *(Wajib menggunakan nama service internal ini, bukan localhost/127.0.0.1 karena database berada di lokasi terpisah)*
   *   **Database Name:** `slims_db`
   *   **Database Username:** `slims_user`
   *   **Database Password:** `slimspassword`

3. Selesaikan pengisian kredensial admin dan klik **Run the Installation**. Aplikasi perpustakaan SLiMS Anda telah sukses beroperasi penuh secara *High Availability* di kluster multi-PC.
