# Ingress

## Deploy Traefik
```
kubectl apply -f traefik-clusterrole.yaml
kubectl apply -f traefik-loadbalancer.yaml
kubectl apply -f treafik-dashboard.yaml
```
## Patch
```
kubectl patch svc traefik-ingress-service -p '{"spec": {"type": "LoadBalancer", "externalIPs":["192.168.11.111"]}}' -n kube-system
```

## kube-prometheus
```
echo '{
  "CN": "traefik.rasp-cluster.local",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "Australia",
      "L": "Sydney",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "NSW"
    }
  ]
}' > ./keys/traefik-csr.json
```

```
cfssl gencert \
  -ca=./keys/ca.pem \
  -ca-key=./keys/ca-key.pem \
  -config=./keys/ca-config.json \
  -profile=kubernetes \
  ./keys/traefik-csr.json | cfssljson -bare traefik-dashboard
cfssl gencert -initca ./keys/traefik-csr.json | cfssljson -bare traefik
mv traefik* ./keys/
```

```
openssl genrsa -out ./keys/traefik-dashboard-key.pem 2048

openssl req -new -key ./keys/traefik-dashboard-key.pem -sha256 \
  -subj "/C=JP/ST=Tokyo/L=hoge/O=kubernetes/CN=traefik-dashboard.rasp-cluster.local" \
  -out ./keys/traefik-dashboard-csr.pem

openssl x509 -req -days 365 -sha256 \
  -in ./keys/traefik-dashboard-csr.pem \
  -signkey ./keys/traefik-dashboard-key.pem \
  -out ./keys/traefik-dashboard.pem

openssl genrsa -out ./keys/grafana-key.pem 2048

openssl req -new -key ./keys/grafana-key.pem -sha256 \
  -subj "/C=JP/ST=Tokyo/L=hoge/O=kubernetes/CN=grafana.rasp-cluster.local" \
  -out ./keys/grafana-csr.pem

openssl x509 -req -days 365 -sha256 \
  -in ./keys/grafana-csr.pem \
  -signkey ./keys/grafana-key.pem \
  -out ./keys/grafana.pem
```


```
sed "s/CRT-traefik/`cat ./keys/traefik-dashboard.pem|base64 -w0`/g" ./yaml/traefik-secret.yaml | \
sed "s/KEY-traefik/`cat ./keys/traefik-dashboard-key.pem|base64 -w0`/g" | \
sed "s/CRT-grafana/`cat ./keys/grafana.pem|base64 -w0`/g" | \
sed "s/KEY-grafana/`cat ./keys/grafana-key.pem|base64 -w0`/g" | \
kubectl apply -f -
```



