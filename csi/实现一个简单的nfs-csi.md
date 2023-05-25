# nfs-csi

## 注册过程

csi插件注册过程只会调用CSI进程的两个方法，这个两个方法分别是`IdentityServer`下的`GetPluginInfo`方法和`NodeServer`下的`NodeGetInfo`方法，我们先实现一下这两个方法验证下注册过程（其他方法暂时均返回error）：

`IdentityServer`下的`GetPluginInfo`方法

```go
func (id *IdentityServer) GetPluginInfo(ctx context.Context, request *csi.GetPluginInfoRequest) (*csi.GetPluginInfoResponse, error) {
    
	return &csi.GetPluginInfoResponse{
		Name: id.nfs.name,
		VendorVersion: id.nfs.version,
	}, nil
}
```

`GetPluginInfo` 接口返回驱动的名称和版本信息，比如 ceph-csi 的名称：`rbd.csi.ceph.com`，该名称与 StorageClass yaml 中的 `provisioner` 字段对应：

```yaml
---
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
     name: csi-rbd-sc
     provisioner: rbd.csi.ceph.com
  parameters:
       ...
  reclaimPolicy: Delete
  allowVolumeExpansion: true
  mountOptions:
     - discard
```





`NodeServer`下的`NodeGetInfo`方法

```
```





## 注册阶段

`IdentityServer`下的`GetPluginInfo`方法

`NodeServer`下的`NodeGetInfo`方法