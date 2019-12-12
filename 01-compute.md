# 01-compute

```
sudo rm -rf /var/lib/clusterctrl/nfs/p1/*
sudo rm -rf /var/lib/clusterctrl/nfs/p2/*
sudo rm -rf /var/lib/clusterctrl/nfs/p3/*
```

sudo vi /etc/dhcpcd.conf

sudo vi /etc/hostname

sudo vi /etc/hosts

```
sudo rm /etc/localtime
sudo rm /etc/timezone
sudo sh -c 'echo Australia/Sydney > /etc/timezone'
sudo dpkg-reconfigure -f noninteractive tzdata
```

```
sudo apt-get update
sudo apt-get upgrade
```

## aaa
```
wget -P ~pi/ \
https://dist2.8086.net/clusterctrl/usbboot/buster/2019-09-26/ClusterCTRL-2019-09-26-lite-1-usbboot.tar.xz
```
```
sudo tar -axf ~pi/ClusterCTRL-2019-09-26-lite-1-usbboot.tar.xz -C /var/lib/clusterctrl/nfs/p1/
sudo usbboot-init 1
sudo touch /var/lib/clusterctrl/nfs/p1/boot/ssh
clusterctrl on p1
```
```
sudo tar -axf ~pi/ClusterCTRL-2019-09-26-lite-1-usbboot.tar.xz -C /var/lib/clusterctrl/nfs/p2/
sudo usbboot-init 2
sudo touch /var/lib/clusterctrl/nfs/p2/boot/ssh
clusterctrl on p2
```
```
sudo tar -axf ~pi/ClusterCTRL-2019-09-26-lite-1-usbboot.tar.xz -C /var/lib/clusterctrl/nfs/p3/
sudo usbboot-init 3
sudo touch /var/lib/clusterctrl/nfs/p3/boot/ssh
clusterctrl on p3
```
## Reference: NAT
```
sudo iptables -t nat -A POSTROUTING -s 172.19.181.1/24 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 172.19.181.2/24 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 172.19.181.3/24 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 172.19.181.4/24 -j MASQUERADE
```
### SSH
```
sudo iptables -t nat -A OUTPUT -p tcp --dport 10022 -d 192.168.11.100 -j DNAT --to-destination 172.19.181.1:22
sudo iptables -t nat -A OUTPUT -p tcp --dport 20022 -d 192.168.11.100 -j DNAT --to-destination 172.19.181.2:22
sudo iptables -t nat -A OUTPUT -p tcp --dport 30022 -d 192.168.11.100 -j DNAT --to-destination 172.19.181.3:22
sudo iptables -t nat -A OUTPUT -p tcp --dport 40022 -d 192.168.11.100 -j DNAT --to-destination 172.19.181.4:22
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```

### etcd - Listen
```
iptables -t nat -A PREROUTING -d 192.168.11.100 -p tcp --dport 12379 -j DNAT --to 172.19.180.1:22
iptables -t nat -A PREROUTING -d 192.168.11.100 -p tcp --dport 22379 -j DNAT --to 172.19.180.2:22
iptables -t nat -A PREROUTING -d 192.168.11.100 -p tcp --dport 32379 -j DNAT --to 172.19.180.3:22
iptables -t nat -A PREROUTING -d 192.168.11.100 -p tcp --dport 42379 -j DNAT --to 172.19.180.4:22
sudo iptables-save > /etc/iptables/rules.v4
```

### Kubernetes - apiserver
```
iptables -t nat -A PREROUTING -d 192.168.11.100 -p tcp --dport 16443 -j DNAT --to 172.19.180.1:22
iptables -t nat -A PREROUTING -d 192.168.11.100 -p tcp --dport 26443 -j DNAT --to 172.19.180.2:22
iptables -t nat -A PREROUTING -d 192.168.11.100 -p tcp --dport 36443 -j DNAT --to 172.19.180.3:22
iptables -t nat -A PREROUTING -d 192.168.11.100 -p tcp --dport 46443 -j DNAT --to 172.19.180.4:22
sudo iptables-save > /etc/iptables/rules.v4
```

## Reference: iptables flush
```
iptables -t nat -F
```

```
ifconfig eth0:1 192.168.11.111
iptables -t nat -A POSTROUTING -s 172.19.181.1  -j SNAT --to 192.168.11.111
```

## Edit hosts
```
for instance in 1 2; do
  ssh rasp-controller-${instance} "\
  sudo sh -c \"echo 192.168.11.111 rasp-worker-1 >> /etc/hosts\"
  sudo sh -c \"echo 192.168.11.112 rasp-worker-2 >> /etc/hosts\"
  sudo sh -c \"echo 192.168.11.113 rasp-worker-3 >> /etc/hosts\"
  "
done
```


