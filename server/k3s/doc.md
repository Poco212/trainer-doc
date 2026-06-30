
## preparation

untuk server 1 dan 2 pastikan jamnya real time

```
sudo timedatectl set-ntp true
```
```
sudo timedatectl set-timezone Asia/Jakarta
```

### manager (server 1)
```
sudo nvim /etc/resolv.conf
```
isi
```
nameserver 8.8.8.8
nameserver 1.1.1.1
```

```
curl -sfL https://get.k3s.io | sh -s server --disable traefik
```

```
sudo cat /var/lib/rancher/k3s/server/node-token 
```

contoh output

```
K105e0a185e516c047f3d0dfd11f4ff8f27bca128295e44d5712b0584cac9227a5e::server:9e9cf0e62969a2fa9800529c79391976
```
### agent (server 2 dan seterusnya)

```
curl -sfL https://get.k3s.io | K3S_URL="https://ip_server:6443" K3S_TOKEN="PASTE_TOKEN_DARI_SERVER" sh -s - agent 
```

### cek sudah konek atau belum

lalukan ini di server 1 yaa

```
 sudo k3s kubectl get nodes   
```

contoh output

```
NAME      STATUS   ROLES           AGE     VERSION
monitor   Ready    <none>          87s     v1.35.5+k3s1
server1   Ready    control-plane   7m41s   v1.35.5+k3s1
```
### bikin label pada setiap agent
```
sudo kubectl label node [hostname_server] role=[rolenya]
```
