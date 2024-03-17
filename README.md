
### 动态卷制备
动态卷制备的实现基于 storage.k8s.io API 组中的 *StorageClass* API 对象。 集群管理员可以根据需要定义多个 StorageClass 对象，每个对象指定一个卷插件（又名 provisioner）， 卷插件向卷制备商提供在创建卷时需要的数据卷信息及相关参数。

集群管理员可以在集群中定义和公开多种存储（来自相同或不同的存储系统），每种都具有自定义参数集。 该设计也确保终端用户不必担心存储制备的复杂性和细微差别，但仍然能够从多个存储选项中进行选择。

### CSI
*VolumeAttachment* 抓取将指定卷挂接到指定节点或从指定节点解除挂接指定卷的意图。

*CSIDriver* 抓取集群上部署的容器存储接口（CSI）卷驱动有关的信息。 Kubernetes 挂接/解除挂接控制器使用此对象来决定是否需要挂接。 Kubelet 使用此对象决定挂载时是否需要传递 Pod 信息。 CSIDriver 对象未划分命名空间。

*CSINode* 包含节点上安装的所有 CSI 驱动有关的信息。CSI 驱动不需要直接创建 CSINode 对象。 只要这些驱动使用 node-driver-registrar 边车容器，kubelet 就会自动为 CSI 驱动填充 CSINode 对象， 作为 kubelet 插件注册操作的一部分。CSINode 的名称与节点名称相同。 如果不存在此对象，则说明该节点上没有可用的 CSI 驱动或 Kubelet 版本太低无法创建该对象。 CSINode 包含指向相应节点对象的 OwnerReference。

*CSIStorageCapacity* 存储一个 CSI GetCapacity 调用的结果。 对于给定的 StorageClass，此结构描述了特定拓扑段中可用的容量。 当考虑在哪里实例化新的 PersistentVolume 时可以使用此项。

### 安装
```bash
velero install \
	--kubeconfig=$HOME/.kubeconfig \
    --provider aws \
	--use-node-agent \
    --plugins velero/velero-plugin-for-aws:v1.2.1,velero/velero-plugin-for-csi:v0.7.0 \
    --features=EnableCSI \
    --bucket velero \
    --secret-file ./credentials-velero \
    --use-volume-snapshots=true \
    --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://10.168.1.201:9000
```

```bash
kubectl delete namespace/velero clusterrolebinding/velero
kubectl delete crds -l component=velero
```

### 备份
```bash
velero backup create clusterall-`date '+%Y%m%d%H%M%S'` --snapshot-move-data
```

###  恢复
```bash
velero restore create clusterall-`date '+%Y%m%d%H%M%S'` --from-backup clusterall-20240317-135114
```

