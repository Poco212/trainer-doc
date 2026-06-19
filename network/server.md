# create static ip for server 1

```
sudo nvim /etc/systemd/network/20-ethernet.network
```

```
[Match]
Type=ether
# Exclude virtual Ethernet interfaces
Kind=!*

[Link]
RequiredForOnline=routable

[Network]
Address=[ip address server]/CIDR
Gateway=[ip address gateway]
DNS=1.1.1.1 8.8.8.8
MulticastDNS=yes

# systemd-networkd does not set per-interface-type default route metrics
# https://github.com/systemd/systemd/issues/17698
# Explicitly set route metric, so that Ethernet is preferred over Wi-Fi and Wi-Fi is preferred over mobile broadband.
# Use values from NetworkManager. From nm_device_get_route_metric_default in
# https://gitlab.freedesktop.org/NetworkManager/NetworkManager/-/blob/main/src/core/devices/nm-device.c
[DHCPv4]
RouteMetric=100

[IPv6AcceptRA]
RouteMetric=100
```

# create static ip for server 2

```
sudo nvim /etc/systemd/network/20-ethernet.network
```

```
[Match]
Type=ether
# Exclude virtual Ethernet interfaces
Kind=!*

[Link]
RequiredForOnline=routable

[Network]
Address=[ip address server]/CIDR
Gateway=[ip address gateway]
DNS=1.1.1.1 8.8.8.8
MulticastDNS=yes

# systemd-networkd does not set per-interface-type default route metrics
# https://github.com/systemd/systemd/issues/17698
# Explicitly set route metric, so that Ethernet is preferred over Wi-Fi and Wi-Fi is preferred over mobile broadband.
# Use values from NetworkManager. From nm_device_get_route_metric_default in
# https://gitlab.freedesktop.org/NetworkManager/NetworkManager/-/blob/main/src/core/devices/nm-device.c
[DHCPv4]
RouteMetric=100

[IPv6AcceptRA]
RouteMetric=100
```
# create systemd ip broadcast for server 1
## install package 
```
sudo pacman -S hostapd
```

## cek interface
```
ip link
```

```
sudo nvim /etc/hostapd/hostapd.conf
```

isi
```
interface=[interface_wireless] example: wlan0
driver=nl80211
ssid=[nama_wifi] example: MyArchAP
hw_mode=g
channel=7
auth_algs=1
wpa=2
wpa_passphrase=[password_wifi] example: 12345678 (minimal 8 character)
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

```
sudo nvim /etc/systemd/network/02-wireless-ap.network
```
isi
```
[Match]
Name=[wireless interface]

[Network]
Address=[ip address]/CIDR
DHCPServer=yes
```
```
systemctl restart systemd-networkd
```
```
sudo systemctl enable --now systemd-networkd
```
```
sudo systemctl enable --now hostapd
```

```
sudo nvim /etc/sysctl.d/99-custome.conf
```
isi
```
net.ipv4.ip_forward=1
```

```
sudo sysctl --system
```

```
reboot
```
# create systemd ip broadcast for server 2
## install package 
```
sudo pacman -S hostapd
```

## cek interface
```
ip link
```

```
sudo nvim /etc/hostapd/hostapd.conf
```

isi
```
interface=[interface_wireless] example: wlan0
driver=nl80211
ssid=[nama_wifi] example: MyArchAP
hw_mode=g
channel=7
auth_algs=1
wpa=2
wpa_passphrase=[password_wifi] example: 12345678 (minimal 8 character)
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

```
sudo nvim /etc/systemd/network/02-wireless-ap.network
```
isi
```
[Match]
Name=[wireless interface]

[Network]
Address=[ip address]/CIDR
DHCPServer=yes
```

```
sudo systemctl restart systemd-networkd
```

```
sudo systemctl enable --now systemd-networkd
```

```
sudo systemctl enable --now hostapd
```

```
sudo nvim /etc/sysctl.d/99-custome.conf
```

isi
```
net.ipv4.ip_forward=1
```

```
sudo sysctl --system
```

```
reboot
```

### note
> ip address server dan admin harus dalam satu network contoh:  
> ip server : 192.168.1.12  
> maka  
> ip admin : 192.168.1.13  
> ip operator : 192.168.1.14  
> ip gateway : 192.168.1.1  
> dan notasi CIDR harus sama yakni 24

## firewall server 1
```bash
sudo firewall-cmd --permanent --new-zone=admin
```
```
sudo firewall-cmd --permanent --zone=admin --add-source=[ip_admin]
```
```bash
sudo firewall-cmd --permanent --zone=admin --add-service=ssh
```
```
sudo firewall-cmd --permanent --zone=admin --add-port=3306/tcp
```
```
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="[ip_server2]" port port="3306" protocol="tcp" accept'
```
example
```
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="10.10.2.4" port port="3306" protocol="tcp" accept'
```
```
sudo firewall-cmd --reload
```
## firewall server 2
```
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
```
```
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="[ip_admin]" port port="22" protocol="tcp" accept'
```
example
```
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="172.27.5.3" port port="22" protocol="tcp" accept'
```
```
sudo firewall-cmd --reload
```
