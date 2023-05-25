## 测试用例

1. 静态盘/动态盘的 pvc 创建/删除（动态盘支持回收后端存储）
2. runc/kata 环境作为文件系统的挂载/卸载，同一块盘能够多次挂载
3. runc/kata 环境作为文件系统挂载后，能够在线扩容并在 pod 内生效

## 操作

### 静态盘  创建/删除

#### 创建volume

```sh
fs_vol=$(openstack volume create --size 1 csi-fs-vol01 -f value -c id)
blk_vol=$(openstack volume create --size 1 csi-blk-vol01 -f value -c id)
```
#### 使用上述volume uuid 替换如下yaml中的变量

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fspv01
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  #persistentVolumeReclaimPolicy: Recycle
  storageClassName: chinaunicom.cinder.sc
  csi:
    driver: chinaunicom.cinder.csi
    volumeHandle: ${fs_vol}
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: blkpv01
spec:
  capacity:
    storage: 1Gi
  volumeMode: Block
  accessModes:
    - ReadWriteOnce
  #persistentVolumeReclaimPolicy: Recycle
  storageClassName: chinaunicom.cinder.sc
  csi:
    driver: chinaunicom.cinder.csi
    volumeHandle: ${blk_vol}
```

## **创建PVC绑定PV**

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: fspvc01
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: chinaunicom.cinder.sc
  volumeName: fspv01
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: blkpvc01
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: chinaunicom.cinder.sc
  volumeName: blkpv01
```



```sh
# kubectl get pvc
NAME      STATUS   VOLUME                     CAPACITY   ACCESS MODES   STORAGECLASS            AGE
fspvc01    Bound     fspv01                   1Gi        RWO            chinaunicom.cinder.sc   4s
blkpvc01   Bound     blkpv01                  1Gi        RWO            chinaunicom.cinder.sc   4s

```



## **kata pod创建关联PVC**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kata-cinder-01
spec:
  runtimeClassName: kata
  volumes:
    - name: fspvc01
      persistentVolumeClaim:
        claimName: fspvc01
    - name: blkpvc01
      persistentVolumeClaim:
        claimName: blkpvc01
  containers:
    - name: kata-cinder-01
      image: myregistry.com/library/busybox
      command: ["sleep", "60000"]
      volumeMounts:
        - mountPath: "/usr/share/fspvc01"
          name: fspvc01
      volumeDevices:
        - devicePath: /dev/rbdblock
          name: blkpvc01

```



验证

```sh
kubectl exec -it kata-cinder-01 -- df -hT
kubectl exec -it kata-cinder-01 -- ls /dev/rbdblock
```

