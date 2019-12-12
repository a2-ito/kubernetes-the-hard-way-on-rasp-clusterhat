# 12-docker-private-registy

## Create Certificate
```
openssl genrsa -out ./keys/docker-registry-key.pem 2048

openssl req -new -key ./keys/docker-registry-key.pem -sha256 \
  -subj "/C=JP/ST=Tokyo/L=hoge/O=kubernetes/CN=docker-registry.rasp-cluster.local" \
  -out ./keys/docker-registry-csr.pem

openssl x509 -req -days 365 -sha256 \
  -in ./keys/docker-registry-csr.pem \
  -signkey ./keys/docker-registry-key.pem \
  -out ./keys/docker-registry.pem
```

## Create Namespace
```
kubectl create namespace docker-registry
```

## Create Security Resource
```
kubectl create secret docker-registry secret-docker-registry -n docker-registry \
 --docker-server=$DOCKER_REGISTRY_SERVER\
 --docker-username=$DOCKER_USER\
 --docker-password=$DOCKER_PASSWORD\
 --docker-email=$DOCKER_EMAIL
```
```
sed "s/CRT-traefik/`cat ./keys/traefik-dashboard.pem|base64 -w0`/g" ./yaml/traefik-secret.yaml | \
sed "s/KEY-traefik/`cat ./keys/traefik-dashboard-key.pem|base64 -w0`/g" | \
sed "s/CRT-grafana/`cat ./keys/grafana.pem|base64 -w0`/g" | \
sed "s/KEY-grafana/`cat ./keys/grafana-key.pem|base64 -w0`/g" | \
sed "s/CRT-docker-registry/`cat ./keys/docker-registry.pem|base64 -w0`/g" | \
sed "s/KEY-docker-registry/`cat ./keys/docker-registry-key.pem|base64 -w0`/g" | \
kubectl apply -f -
```
```
kubectl delete pod -n kube-system $(kubectl get pod -n kube-system | grep traefik | cut -d' ' -f1)
```

### docker-registry
```
sed "s/CRT/`cat ./keys/docker-registry.pem|base64 -w0`/g" ./yaml/docker-registry-secret.yaml | \
sed "s/KEY/`cat ./keys/docker-registry-key.pem|base64 -w0`/g" | \
kubectl apply -f -

kubectl create secret tls docker-registry-cert \
  -n docker-registry \
  --key ./keys/docker-registry-key.pem \
  --cert ./keys/docker-registry.pem
```

## Create Configmap
```
kubectl apply -f yaml/docker-registy-configmap.yaml
```

## Create Deployment & Service
```
kubectl apply -f yaml/docker-registry-deployment.yaml
```

## Create Ingress
```
kubectl apply -f yaml/docker-registry-ingress.yaml
```

## Client Configuration

```
sudo mkdir /etc/docker/certs.d/docker-registry.rasp-cluster.local
```
### Create Client Certificate
```
openssl genrsa -out client.key 4096
openssl req -new -x509 -text -key client.key -out client.cert
mv client.key client.cert /etc/docker/certs.d/docker-registry.rasp-cluster.local/
```

```ca.crt``` is also required to be deployed on the directory.

### Verification
```
docker pull ubuntu:16.04
docker tag ubuntu:16.04 docker-registry.rasp-cluster.local
docker push docker-registry.rasp-cluster.local/ubuntu:16.04
```
