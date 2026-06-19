# create static ip for server 1

```
nvim /etc/systemd/network/20-ethernet.network
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
nvim /etc/systemd/network/20-ethernet.network
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
pacman -S hostapd
```

## cek interface
```
ip link
```

```
nvim /etc/hostapd/hostapd.conf
```

isi
```
interface=wlan0
driver=nl80211
ssid=MyArchAP
hw_mode=g
channel=7
auth_algs=1
wpa=2
wpa_passphrase=MySecurePassword
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

```
nvim /etc/systemd/network/02-wireless-ap.network
```

```
systemctl restart systemd-networkd
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
sudo systemctl enable --now systemd-networkd
```

```
nvim /etc/sysctl.d/30-ipforward.conf
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

```
sudo systemctl enable --now hostapd
```
# create systemd ip broadcast for server 1
## install package 
```
pacman -S hostapd
```

## cek interface
```
ip link
```

```
nvim /etc/hostapd/hostapd.conf
```

isi
```
interface=wlan0
driver=nl80211
ssid=MyArchAP
hw_mode=g
channel=7
auth_algs=1
wpa=2
wpa_passphrase=MySecurePassword
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

```
nvim /etc/systemd/network/02-wireless-ap.network
```

```
systemctl restart systemd-networkd
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
sudo systemctl enable --now systemd-networkd
```

```
nvim /etc/sysctl.d/30-ipforward.conf
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

```
sudo systemctl enable --now hostapd
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
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="[ip_network_client]/24" port port="80" protocol="tcp" accept'
```
example
```
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.2.0/24" port port="80" protocol="tcp" accept'
```
> [NOTE]
> angka terakhir pada ip harus ditulis `0` 
```
sudo firewall-cmd --reload
```
