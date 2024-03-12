```bash
$ kubectl get volumesnapshotclasses.snapshot.storage.k8s.io 
$ kubectl get volumesnapshots.snapshot.storage.k8s.io 
$ kubectl get volumesnapshotcontents.snapshot.storage.k8s.io
$ kubectl get pods --all-namespaces -o=jsonpath='{range .items[*]}{"\n"}{range .spec.containers[*]}{.image}{", "}{end}{end}' | grep snapshot-controller
quay.io/k8scsi/snapshot-controller:v2.0.1, 
```

```bash
# Change to the latest supported snapshotter release branch
$ SNAPSHOTTER_BRANCH=release-6.3

# Apply VolumeSnapshot CRDs
$ kubectl apply -f snapshot.storage.k8s.io_volumesnapshotclasses.yaml
$ kubectl apply -f snapshot.storage.k8s.io_volumesnapshotcontents.yaml
$ kubectl apply -f snapshot.storage.k8s.io_volumesnapshots.yaml

# Change to the latest supported snapshotter version
$ SNAPSHOTTER_VERSION=v6.3.3

# Create snapshot controller
$ kubectl apply -f rbac-snapshot-controller.yaml
$ kubectl apply -f setup-snapshot-controller.yaml
```





```bash
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/snapshot-controller:v6.3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/csi-snapshotter:v4.2.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/csi-resizer:v1.3.0
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/csi-provisioner:v3.0.0
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/csi-node-driver-registrar:v2.3.0
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/csi-attacher:v3.3.0

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/snapshot-controller:v6.3.1 registry.cn-hangzhou.aliyuncs.com/google_containers/snapshot-controller:v6.3.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/csi-snapshotter:v4.2.1 registry.cn-hangzhou.aliyuncs.com/google_containers/csi-snapshotter:v4.2.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/csi-resizer:v1.3.0 registry.cn-hangzhou.aliyuncs.com/google_containers/csi-resizer:v1.3.0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/csi-provisioner:v3.0.0 registry.cn-hangzhou.aliyuncs.com/google_containers/csi-provisioner:v3.0.0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/csi-node-driver-registrar:v2.3.0 registry.cn-hangzhou.aliyuncs.com/google_containers/csi-node-driver-registrar:v2.3.0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/csi-attacher:v3.3.0 registry.cn-hangzhou.aliyuncs.com/google_containers/csi-attacher:v3.3.0
```

```bash
kubectl apply -f snapshot.storage.k8s.io_volumesnapshotclasses.yaml
kubectl apply -f snapshot.storage.k8s.io_volumesnapshotcontents.yaml
kubectl apply -f snapshot.storage.k8s.io_volumesnapshots.yaml

kubectl apply -f rbac-snapshot-controller.yaml
kubectl apply -f setup-snapshot-controller.yaml
```

```bash
git clone https://github.com/SynologyOpenSource/synology-csi.git
cp config/client-info-template.yml config/client-info.yml
vim config/client-info.yml
```


```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
  name: synostorage
provisioner: csi.san.synology.com
parameters:
  fsType: 'btrfs'
  dsm: '10.168.1.201'
  location: '/volume1/k8s-csi-vol'
  formatOptions: '--nodiscard'
reclaimPolicy: Delete
allowVolumeExpansion: true
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-logs
  namespace: nginx-example
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: synostorage
```

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: new-snapshot-test
  namespace: nginx-example
spec:
  volumeSnapshotClassName: synology-snapshotclass
  source:
    persistentVolumeClaimName: nginx-logs
```
