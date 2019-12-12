# Bootstrapping an H/A Kubernetes Control Plane

## Install etcd binaries
```
for instance in 1 2 3; do
  ssh p${instance} "\
    sudo mkdir -p /etc/etcd /var/lib/etcd
    sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
    wget https://raw.githubusercontent.com/robertojrojas/kubernetes-the-hard-way-raspberry-pi/master/etcd/etcd-3.1.5-arm.tar.gz
    tar xzf etcd-3.1.5-arm.tar.gz
    sudo mv etcd-3.1.5-arm/etcd* /usr/local/bin/
    "
done
```

## Configure the etcd Server
```
for instance in 1 2 3; do
cat > etcd.service <<"EOF"
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Environment=ETCD_UNSUPPORTED_ARCH=arm
Type=notify
ExecStart=/usr/local/bin/etcd \
  --name ETCD_NAME \
  --cert-file=/etc/etcd/kubernetes.pem \
  --key-file=/etc/etcd/kubernetes-key.pem \
  --peer-cert-file=/etc/etcd/kubernetes.pem \
  --peer-key-file=/etc/etcd/kubernetes-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ca.pem \
  --peer-client-cert-auth \
  --client-cert-auth \
  --initial-advertise-peer-urls https://INTERNAL_IP:2380 \
  --listen-peer-urls https://INTERNAL_IP:2380 \
  --listen-client-urls https://INTERNAL_IP:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://INTERNAL_IP:2379 \
  --initial-cluster-token etcd-cluster \
  --initial-cluster p1=https://192.168.11.101:2380,p2=https://192.168.11.102:2380,p3=https://192.168.11.103:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
  export INTERNAL_IP=192.168.11.10${instance}
  export ETCD_NAME=p${instance}
  sed -i s/INTERNAL_IP/${INTERNAL_IP}/g etcd.service
  sed -i s/ETCD_NAME/${ETCD_NAME}/g etcd.service
  scp etcd.service p${instance}:~
  rm etcd.service
  ssh p${instance} "sudo mv etcd.service /etc/systemd/system/"
done
```

```
for instance in 1 2 3; do
  ssh p${instance} "\
    sudo systemctl daemon-reload
    sudo systemctl enable etcd
    sudo systemctl start etcd &
  "
done
```
```
for instance in 1 2 3; do
  ssh p${instance} "\
  sudo ETCDCTL_API=3 etcdctl member list \
    --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/etcd/ca.pem \
    --cert=/etc/etcd/kubernetes.pem \
    --key=/etc/etcd/kubernetes-key.pem
  "
done
```
