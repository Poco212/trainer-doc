# create static ip ethernet for server 1

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

# create static ip ethernet for server 2

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
>[Note]
>IP server 1 dan Ip server harus 1 network
# create systemd ip atau router  for server 1
## install package 
# create systemd ip broadcast for server 2
## install package 

### note
> ip address server dan admin harus dalam satu network contoh:  
> ip server : 192.168.1.12  
> maka  
> ip admin : 192.168.1.13  
> ip operator : 192.168.1.14  
> ip gateway : 192.168.1.1  
> dan notasi CIDR harus sama yakni 24

## firewall server 1
```
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="[ip_server2]" port port="port_apk" protocol="tcp" accept'
```
```
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="[ip_admin]" port port="22" protocol="tcp" accept'
```
example
```
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="10.10.2.4" port port="8080" protocol="tcp" accept'
```
```
sudo firewall-cmd --reload
```
## firewall server 2

```
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="[ip_network_client]/24" port port="port_apk" protocol="tcp" accept'
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
