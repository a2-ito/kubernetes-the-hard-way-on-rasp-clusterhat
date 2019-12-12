# Kubernetes the hard way
## Copy Kubeconfigs
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  scp kube-proxy.kubeconfig ${instance}.kubeconfig ${instance}:~/
done
```
## Copy Keys
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  scp ./keys/ca.pem ./keys/${instance}.pem ./keys/${instance}-key.pem ${instance}:~/
done
```

## Provisioning a Kubernetes Worker Node
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
	sudo apt-get update
	sudo apt-get -y install socat conntrack ipset
  "
done
```
## Disable Swap
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo swapoff -a
    sudo swapon --show
  "
done
```

## Download and Install Worker Binaries
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    curl -sSL http://get.docker.com  | sh
    sudo usermod -aG docker pi
    wget -q --show-progress --https-only --timestamping \
      https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-arm-v0.6.0.tgz \
      https://storage.googleapis.com/kubernetes-release/release/v1.15.5/bin/linux/arm/kubectl \
      https://storage.googleapis.com/kubernetes-release/release/v1.15.5/bin/linux/arm/kube-proxy \
      https://storage.googleapis.com/kubernetes-release/release/v1.15.5/bin/linux/arm/kubelet
    "
done
```
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    wget -q --show-progress --https-only --timestamping \
      https://github.com/opencontainers/runc/releases/download/v1.0.0-rc8/runc.arm \
      http://mirror.archlinuxarm.org/arm/community/containerd-1.3.1-1-arm.pkg.tar.xz \
      https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.15.0/crictl-v1.15.0-linux-arm.tar.gz \
      https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-arm-v0.6.0.tgz \
      https://storage.googleapis.com/kubernetes-release/release/v1.15.5/bin/linux/arm/kubectl \
      https://storage.googleapis.com/kubernetes-release/release/v1.15.5/bin/linux/arm/kube-proxy \
      https://storage.googleapis.com/kubernetes-release/release/v1.15.5/bin/linux/arm/kubelet
    "
done
scp runc.arm *.tgz *.xz kube* 192.168.11.112:~
scp runc.arm *.tgz *.xz kube* 192.168.11.113:~
```
### Create the installation directories
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo mkdir -p \
      /etc/cni/net.d \
      /opt/cni/bin \
      /var/lib/kubelet \
      /var/lib/kube-proxy \
      /var/lib/kubernetes \
      /var/run/kubernetes
    "
done
```
### Install the worker binaries
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo tar -xvf cni-plugins-arm-v0.6.0.tgz -C /opt/cni/bin/
    chmod +x crictl kubectl kube-proxy kubelet 
    sudo mv crictl kubectl kube-proxy kubelet /usr/local/bin/
  "
done
```
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    tar Jxf containerd-1.3.1-1-arm.pkg.tar.xz
    sudo mv usr/bin/* /usr/local/bin/
    sudo tar -xvf cni-plugins-arm-v0.6.0.tgz -C /opt/cni/bin/
	#sudo mv runc.arm runc
    tar xzf crictl-v1.15.0-linux-arm.tar.gz
    chmod +x crictl kubectl kube-proxy kubelet runc
    sudo mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
  "
done
```

### DNS
<details>

```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
  sudo mkdir -p /run/systemd/resolve/
  sudo ln -s /etc/resolv.conf /run/systemd/resolve/resolv.conf
  "
done
```

</details>

## Configure CNI Networking
<details>

### Retrieve the Pod CIDR range for the current compute instance
### Create the ```bridge``` network configuration file

```
for instance in 1 2 3; do
POD_CIDR=10.200.${instance}.0/24
cat <<EOF | tee 10-bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
  scp 10-bridge.conf rasp-worker-${instance}:~
  ssh rasp-worker-${instance} "\
    sudo mv 10-bridge.conf /etc/cni/net.d/
  "
  rm 10-bridge.conf
done
```

### Create the ```loopback``` network configuration file
```
cat <<EOF | tee 99-loopback.conf
{
    "cniVersion": "0.3.1",
    "name": "lo",
    "type": "loopback"
}
EOF
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  scp 99-loopback.conf ${instance}:~
  ssh ${instance} "\
    sudo mv 99-loopback.conf /etc/cni/net.d/
  "
done
rm 99-loopback.conf
```

</details>

## Configure containerd
<details>

### Create the ```containerd``` configuration file
```
cat << EOF | tee config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/sbin/runc"
      runtime_root = ""
EOF
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  scp config.toml ${instance}:~
  ssh ${instance} "\
    sudo mkdir -p /etc/containerd/
    sudo mv config.toml /etc/containerd/
  "
done
rm config.toml
```

### Create the ```containerd.service``` systemd unit file
```
cat <<EOF | tee containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  scp containerd.service ${instance}:~
  ssh ${instance} "\
    sudo mv containerd.service /etc/systemd/system/
  "
done
rm containerd.service
```

### Start the Worker Services - containerd
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo systemctl daemon-reload
    sudo systemctl enable containerd
    sudo systemctl start containerd
  "
done
```
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo systemctl status containerd
  "
done
```

</details>

## Configure the Kubelet
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo mv ${instance}-key.pem ${instance}.pem /var/lib/kubelet/
    sudo mv ${instance}.kubeconfig /var/lib/kubelet/kubeconfig
    sudo mv ca.pem /var/lib/kubernetes/
  "
done
```
### Create the ```kubelet-config.yaml``` configurations file
```
for instance in 1 2 3; do
POD_CIDR=10.200.${instance}.0/24
cat <<EOF | tee kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
#resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/rasp-worker-${instance}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/rasp-worker-${instance}-key.pem"
EOF
  scp kubelet-config.yaml rasp-worker-${instance}:~
  ssh rasp-worker-${instance} "\
    sudo mv kubelet-config.yaml /var/lib/kubelet/
  "
rm kubelet-config.yaml
done
```

### Create the ```kubelet.service``` systemd unit file
```
cat > kubelet.service <<"EOF"
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \
  --config=/var/lib/kubelet/kubelet-config.yaml \
  --container-runtime=docker \
  --image-pull-progress-deadline=2m \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --network-plugin=cni \
  --register-node=true \
  --authentication-token-webhook=true \
  --authorization-mode=Webhook \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  scp kubelet.service ${instance}:~
  ssh ${instance} "\
    sudo mv kubelet.service /etc/systemd/system/
  "
done
rm kubelet.service
```

### Start the Worker Services - kubelet
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo systemctl daemon-reload
    sudo systemctl enable kubelet
    sudo systemctl start kubelet
  "
done
```

```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo systemctl status kubelet
  "
done
```


## Configure the Kubernetes Proxy
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
  "
done
```
### Create the ```kube-proxy-config.yaml``` configuration file
```
cat <<EOF | tee kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
iptables:
  masqueradeAll: true
EOF
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  scp kube-proxy-config.yaml ${instance}:~
  ssh ${instance} "\
    sudo mv kube-proxy-config.yaml /var/lib/kube-proxy
  "
done
rm kube-proxy-config.yaml
```
### Create the ```kube-proxy.service``` systemd unit file
```
cat <<EOF | tee kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  scp kube-proxy.service ${instance}:~
  ssh ${instance} "\
    sudo mv kube-proxy.service /etc/systemd/system/
  "
done
rm kube-proxy.service
```

### Start the Worker Services
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo systemctl daemon-reload
    sudo systemctl enable kube-proxy
    sudo systemctl start kube-proxy
  "
done
```
```
for instance in rasp-worker-1 rasp-worker-2 rasp-worker-3; do
  ssh ${instance} "\
    sudo systemctl status kube-proxy
  "
done
```

## Verification
```
for instance in rasp-controller-1 rasp-controller-2; do
  ssh ${instance} "\
    kubectl get nodes --kubeconfig admin.kubeconfig
  "
done
```
```
kubectl run --kubeconfig admin.kubeconfig \
  --generator=run-pod/v1 busybox --image=busybox:1.28 --command -- sleep 3600
```
