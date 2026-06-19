# check interface for ethernet

```
nmcli device status
```

# create admin pc connection

```
nmcli connection add type ethernet ifname [interface] con-name "admin-connection" ipv4.method manual ipv4.addresses [ip address for admin]/[CIDR] ipv4.gateway [ip address gateway] ipv4.dns 8.8.8.8
```


# create client pc connection

```
nmcli connection add type ethernet ifname [interface] con-name "client-connection" ipv4.method manual ipv4.addresses [ip address for client]/[CIDR] ipv4.gateway [ip address gateway] ipv4.dns 8.8.8.8
```

# admin pc

```
echo "nmcli connection up admin-connection" >> /home/admin/.bash_profile
```


# client pc

```
echo "nmcli connection up client-connection" >> /home/client/.bash_profile
```


### note
> ip address server dan admin harus dalam satu network contoh:  
> ip server : 192.168.1.12  
> maka  
> ip admin : 192.168.1.13  
> ip operator : 192.168.1.14  
> ip gateway : 192.168.1.1  
> dan notasi CIDR harus sama yakni 24  
