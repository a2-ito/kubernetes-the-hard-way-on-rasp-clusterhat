# Provisioning a CA and Generationg TLS Certificates


## Certificate Authority

```
cat > ./keys/ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ./keys/ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF
```

```
cfssl gencert -initca ./keys/ca-csr.json | cfssljson -bare ca
mv ./*.pem ./*.csr ./keys/
```
## Create Certificates
```
cat > ./keys/template-csr.json <<EOF
{
  "CN": "1_CN",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "2_C",
      "L": "3_L",
      "O": "4_O",
      "OU": "5_OU",
      "ST": "6_ST"
    }
  ]
}
EOF
```
```
sbj=(
kubernetes';'kubernetes';'US';'Portland';'Kubernetes';'"Kubernetes The Hard Way"';'Oregon';'"10.32.0.1,192.168.11.101,192.168.11.102,192.168.11.103,192.168.11.104,192.168.11.105,127.0.0.1,p1,p2,p3,p4,rasp-master,192.168.11.100"
kube-controller-manager';'system:kube-controller-manager';'US';'Portland';'system:kube-controller-manager';'"Kubernetes The Hard Way"';'Oregon';'""
kube-scheduler';'system:kube-scheduler';'US';'Portland';'system:kube-scheduler';'"Kubernetes The Hard Way"';'Oregon';'""
admin';'admin';'US';'Portland';'system:masters';'"Kubernetes The Hard Way"';'Oregon';'""
service-account';'service-accounts';'US';'Portland';'Kubernetes';'"Kubernetes The Hard Way"';'Oregon';'""
rasp-worker-1';'system:node:rasp-worker-1';'US';'Portland';'system:nodes';'"Kubernetes The Hard Way"';'Oregon';'"rasp-worker-1,192.168.11.111"
rasp-worker-2';'system:node:rasp-worker-2';'US';'Portland';'system:nodes';'"Kubernetes The Hard Way"';'Oregon';'"rasp-worker-2,192.168.11.112"
rasp-worker-3';'system:node:rasp-worker-3';'US';'Portland';'system:nodes';'"Kubernetes The Hard Way"';'Oregon';'"rasp-worker-3,192.168.11.113"
kube-proxy';'system:kube-proxy';'US';'Portland';'system:node-proxier';'"Kubernetes The Hard Way"';'Oregon';'""
)
for array in "${sbj[@]}"
do
  servertype=`echo "${array}" | cut -d';' -f1`
  _1_CN=`echo "${array}" | cut -d';' -f2`
  _2_C=`echo "${array}" | cut -d';' -f3`
  _3_L=`echo "${array}" | cut -d';' -f4`
  _4_O=`echo "${array}" | cut -d';' -f5`
  _5_OU=`echo "${array}" | cut -d';' -f6`
  _6_ST=`echo "${array}" | cut -d';' -f7`
  _7_HN=`echo "${array}" | cut -d';' -f8`
  echo ${_6_ST}
  cat ./keys/template-csr.json |\
  sed -e "s/1_CN/${_1_CN}/g" \
	-e "s/2_C/${_2_C}/g" \
	-e "s/3_L/${_3_L}/g" \
	-e "s/4_O/${_4_O}/g" \
	-e "s/5_OU/${_5_OU}/g" \
	-e "s/6_ST/${_6_ST}/g" > ./keys/${servertype}-csr.json
  cfssl gencert \
    -ca=./keys/ca.pem \
    -ca-key=./keys/ca-key.pem \
    -config=./keys/ca-config.json \
    -profile=kubernetes \
    -hostname="${_7_HN}" \
    ./keys/${servertype}-csr.json | cfssljson -bare ${servertype}
done
mv ./*.pem ./*.csr ./keys/
```

```
for instance in p1 p2 p3;
do
  scp ./keys/ca.pem ./keys/ca-key.pem ./keys/kubernetes-key.pem ./keys/kubernetes.pem \
    ./keys/service-account-key.pem ./keys/service-account.pem ${instance}:~
done
```


