# 12-persistent-volume

## Storage Class

| NFS Sv Directory | PV Name                | accessMode | persistentVolumeReclaimPolicy |
| ---------------- |:-----------------------|:--------------|:--------|
| /test            | pv-nfs-test            | ReadWriteOnce | Recycle |
| /docker-registry | pv-nfs-docker-registry | ReadWriteOnce | Recycle |
| /app             | pv-nfs-app             | ReadWriteOnce | Recycle |

## NFS Configuration

### Test
```
docker run -d --cap-add=SYS_ADMIN \
  --name="nfs4" \
  --net=host \
  -v /home/pi/work/rasp-filesv:/nfsshare \
  -e NFS_EXPORT_DIR_1=/nfsshare \
  -e NFS_EXPORT_DOMAIN_1=\* \
  -e NFS_EXPORT_OPTIONS_1="rw,fsid=0,root_squash,no_subtree_check,insecure" \
  -p 111:111 -p 111:111/udp \
  -p 2049:2049 -p 2049:2049/udp \
  grewhit/armhf-nfs4-server
```

```
sudo mount -v -t nfs4 192.168.11.3:/ /mnt/test/
```

### NFS Server Configuration 
```
docker run -d --cap-add=SYS_ADMIN \
  --name="nfs-for-kubernetes" \
  --net=host \
  -v /mnt/sharerdvol/nfsshare/:/nfsshare \
  -e NFS_EXPORT_DIR_1=/nfsshare \
  -e NFS_EXPORT_DOMAIN_1=\* \
  -e NFS_EXPORT_OPTIONS_1="rw,fsid=0,root_squash,no_subtree_check,insecure" \
  -p 111:111 -p 111:111/udp \
  -p 2049:2049 -p 2049:2049/udp \
  grewhit/armhf-nfs4-server
```

### Verification
```
sudo mount -v -t nfs4 192.168.11.3:/test /mnt/test/
```

```
kubectl apply -f yaml/pv-nfs-test.yaml
kubectl apply -f yaml/pv-nfs-docker-registry.yaml
kubectl apply -f yaml/pv-nfs-app.yaml
```
```
kubectl apply -f yaml/pvc-nfs-test.yaml
kubectl apply -f yaml/pvc-nfs-docker-registry.yaml
kubectl apply -f yaml/pvc-nfs-app.yaml
```


