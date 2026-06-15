# create static ip

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

> ip address server dan admin harus dalam satu network contoh:  
> ip server : 192.168.1.12  
> maka  
> ip admin : 192.168.1.13  
> ip operator : 192.168.1.14  
> ip gateway : 192.168.1.1  
> dan notasi CIDR harus sama yakni 24  
