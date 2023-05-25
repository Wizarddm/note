

## csi注册过程

![图片](https://mmbiz.qpic.cn/mmbiz_png/3Ulmkt3NWasEQ5UQb0kBpP7VMHhjFrddBSXExD9Mlia09nACQq20XdtvuyZGlbPsHrHpQ6rIaGyfQ2fVHw9bibqw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

CSI插件注册过程只会调用CSI进程的两个方法，这两个方法分别是`IdentityServer`下的`GetPluginInfo`方法和`NodeServer`下的`NodeGetInfo`方法

- `IdentityServer`下的`GetPluginInfo`方法

```go
// GetPluginInfo 获取CSI插件的信息，比如名称、版本号
func (ids *IdentityServer) GetPluginInfo(ctx context.Context, req *csi.GetPluginInfoRequest) (*csi.GetPluginInfoResponse, error) {
	klog.V(5).Infof("Using default GetPluginInfo")
	if ids.Name == "" {
		return nil, status.Error(codes.Unavailable, "Driver name not configured")
	}
	if ids.FqVersion == "" {
		return nil, status.Error(codes.Unavailable, "Driver is missing version")
	}
	return &csi.GetPluginInfoResponse{ // 返回默认的插件名称以及版本号
		Name:          ids.Name,
		VendorVersion: ids.FqVersion,
	}, nil
}
```

- `NodeServer`下的`NodeGetInfo`方法

```go
// NodeGetInfo 获取CSI节点信息，比如最大支持卷个数 TODO 这个玩意在我们的场景下需要吗？
func (ns *NodeServer) NodeGetInfo(ctx context.Context, req *csi.NodeGetInfoRequest) (*csi.NodeGetInfoResponse, error) {
   args := []string{"-f"}
   hostname, _, err := util.ExecCommand("hostname", args...)
   if err != nil {
      return nil, fmt.Errorf("Cannot get node fqdn name, error:%v", err)
   }
   hostname = strings.TrimSuffix(hostname, "\n")
   return &csi.NodeGetInfoResponse{
      NodeId: hostname,
   }, nil
}
```



## 注册过程产物

注册过程中会有如下产物：

- node-driver-registrar进程的sock文件：/var/lib/kubelet/plugins_registry/{csiDriverName}-reg.sock
- CSI进程的sock文件：/var/lib/kubelet/plugins/{xxx}/csi.sock
- 节点对应Node对象的annotation中会有一个关于该CSI插件的注解
- 会有一个CSINode对象

## 验证

先观察pod是否正常启动：

```sh
$ k -n host-csi get ds
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR              AGE
csi-nodeplugin-lustre   1         1         1       1            1           <none>                     31d
csi-nodeplugin2         14        14        13      2            13          <none>                     57d
csi-nodeplugin-arm      1         1         1       1            1           kubernetes.io/arch=arm64   57d
csi-nodeplugin          1         1         1       1            1           <none>                     57d
$ k -n host-csi get po -o wide 
NAME                            READY   STATUS             RESTARTS      AGE   IP              NODE       NOMINATED NODE   READINESS GATES
csi-nodeplugin2-nh2jk           0/2     ImagePullBackOff   0             31d   172.31.33.241   master1    <none>           <none>
csi-nodeplugin2-6twpr           2/2     Running            0             57d   172.32.1.3      worker9    <none>           <none>
csi-nodeplugin2-tvpkd           2/2     Running            0             57d   172.31.33.111   worker7    <none>           <none>
csi-nodeplugin2-22cxr           2/2     Running            0             44d   172.32.1.20     worker24   <none>           <none>
csi-nodeplugin2-jvqrb           2/2     Running            0             57d   172.32.1.2      worker8    <none>           <none>
csi-nodeplugin2-cxc9x           2/2     Running            0             57d   172.32.1.19     worker23   <none>           <none>
cinder-csi-controllerplugin-0   5/5     Running            0             16d   10.13.141.91    worker6    <none>           <none>
csi-nodeplugin2-5k9mc           2/2     Running            0             57d   172.31.33.242   worker1    <none>           <none>
csi-nodeplugin2-q4cgg           2/2     Running            0             57d   172.31.33.243   worker2    <none>           <none>
csi-nodeplugin-arm-5bp5s        2/2     Running            0             57d   172.20.19.26    worker25   <none>           <none>
csi-nodeplugin2-2mdvc           2/2     Running            0             13d   172.32.1.7      worker12   <none>           <none>
csi-nodeplugin2-vj7cp           2/2     Running            0             57d   172.32.1.12     worker16   <none>           <none>
csi-nodeplugin2-hdxbf           2/2     Terminating        0             57d   172.32.1.18     worker22   <none>           <none>
lustre-csi-controllerplugin-0   3/3     Running            4 (30d ago)   30d   10.8.182.102    worker3    <none>           <none>
csi-nodeplugin2-87st2           2/2     Running            0             57d   172.31.33.136   worker4    <none>           <none>
csi-nodeplugin2-r7fcx           2/2     Running            0             57d   172.31.33.92    worker5    <none>           <none>
csi-nodeplugin2-98jgm           2/2     Running            0             57d   172.32.1.5      worker10   <none>           <none>
csi-nodeplugin-wtkpc            3/3     Running            0             16d   172.32.1.1      worker6    <none>           <none>
csi-nodeplugin2-cwmrm           2/2     Running            0             57d   172.32.1.6      worker11   <none>           <none>
cubs-csi-controllerplugin-0     3/3     Running            0             16d   10.13.141.93    worker6    <none>           <none>
csi-nodeplugin-lustre-rwb4b     3/3     Running            0             30d   172.31.33.132   worker3    <none>           <none>
```
pod正常启动后，先查看node-driver-registrar的日志，注册正常：

```sh
$ k logs csi-nodeplugin2-r7fcx -n host-csi
Defaulted container "cinder-node-driver-registrar" out of: cinder-node-driver-registrar, csi-plugin
I0308 03:06:42.945792       1 main.go:166] Version: v2.6.0
I0308 03:06:42.945856       1 main.go:167] Running node-driver-registrar in mode=registration
I0308 03:06:43.957072       1 node_register.go:53] Starting Registration Server at: /registration/chinaunicom.cinder.csi-reg.sock
I0308 03:06:43.957475       1 node_register.go:62] Registration Server started at: /registration/chinaunicom.cinder.csi-reg.sock
I0308 03:06:43.959785       1 node_register.go:92] Skipping HTTP server because endpoint is set to: ""
I0308 03:06:44.952593       1 main.go:102] Received GetInfo call: &InfoRequest{}
E0308 03:06:44.952707       1 main.go:107] "Failed to create registration probe file" err="open /var/lib/kubelet/plugins/chinaunicom.cinder.csi/registration: no such file or directory" registrationProbePath="/var/lib/kubelet/plugins/chinaunicom.cinder.csi/registration"
I0308 03:06:44.952757       1 main.go:109] "Kubelet registration probe created" path="/var/lib/kubelet/plugins/chinaunicom.cinder.csi/registration"
I0308 03:06:44.983049       1 main.go:120] Received NotifyRegistrationStatus call: &RegistrationStatus{PluginRegistered:true,Error:,}
I0418 08:07:17.410355       1 main.go:102] Received GetInfo call: &InfoRequest{}
E0418 08:07:17.410435       1 main.go:107] "Failed to create registration probe file" err="open /var/lib/kubelet/plugins/chinaunicom.cinder.csi/registration: no such file or directory" registrationProbePath="/var/lib/kubelet/plugins/chinaunicom.cinder.csi/registration"
I0418 08:07:17.410455       1 main.go:109] "Kubelet registration probe created" path="/var/lib/kubelet/plugins/chinaunicom.cinder.csi/registration"
I0418 08:07:17.417950       1 main.go:120] Received NotifyRegistrationStatus call: &RegistrationStatus{PluginRegistered:true,Error:,}
I0423 01:47:58.997985       1 main.go:102] Received GetInfo call: &InfoRequest{}
E0423 01:47:58.998067       1 main.go:107] "Failed to create registration probe file" err="open /var/lib/kubelet/plugins/chinaunicom.cinder.csi/registration: no such file or directory" registrationProbePath="/var/lib/kubelet/plugins/chinaunicom.cinder.csi/registration"
I0423 01:47:58.998095       1 main.go:109] "Kubelet registration probe created" path="/var/lib/kubelet/plugins/chinaunicom.cinder.csi/registration"
I0423 01:47:59.007750       1 main.go:120] Received NotifyRegistrationStatus call: &RegistrationStatus{PluginRegistered:true,Error:,}
```

再看看自己编码nfs-csi容器日志，和之前分析的一样，注册过程只调用了`GetPluginInfo`和`NodeGetInfo`方法：

```sh
[root@master1 ~]# k logs csi-nodeplugin2-r7fcx -n host-csi -c csi-plugin
I0308 11:06:43.265592       1 main.go:28] Multi CSI Driver Names: chinaunicom.cinder.csi, endPoints: csi.sock
I0308 11:06:43.265865       1 main.go:45] Prepare csi endpoint unix://csi/chinaunicom.cinder.csi/csi.sock
W0308 11:06:43.265955       1 client_config.go:617] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0308 11:06:43.266682       1 util.go:214] Kernel version: 5.17.5
I0308 11:06:43.600554       1 capability.go:12] Enabling controller service capability: LIST_VOLUMES
I0308 11:06:43.600585       1 capability.go:12] Enabling controller service capability: CREATE_DELETE_VOLUME
I0308 11:06:43.600596       1 capability.go:12] Enabling controller service capability: PUBLISH_UNPUBLISH_VOLUME
I0308 11:06:43.600606       1 capability.go:12] Enabling controller service capability: CREATE_DELETE_SNAPSHOT
I0308 11:06:43.600615       1 capability.go:12] Enabling controller service capability: LIST_SNAPSHOTS
I0308 11:06:43.600624       1 capability.go:12] Enabling controller service capability: EXPAND_VOLUME
I0308 11:06:43.600634       1 capability.go:12] Enabling controller service capability: CLONE_VOLUME
I0308 11:06:43.600643       1 capability.go:12] Enabling controller service capability: LIST_VOLUMES_PUBLISHED_NODES
I0308 11:06:43.600651       1 capability.go:12] Enabling controller service capability: GET_VOLUME
I0308 11:06:43.600686       1 capability.go:30] Enabling volume access mode: SINGLE_NODE_WRITER
I0308 11:06:43.600713       1 capability.go:21] Enabling node service capability: STAGE_UNSTAGE_VOLUME
I0308 11:06:43.600730       1 capability.go:21] Enabling node service capability: EXPAND_VOLUME
I0308 11:06:43.600739       1 capability.go:21] Enabling node service capability: GET_VOLUME_STATS
I0308 11:06:43.604864       1 mount_linux.go:218] Cannot run systemd-run, assuming non-systemd OS
I0308 11:06:43.604897       1 mount_linux.go:219] systemd-run output: Failed to create bus connection: No such file or directory
, failed with: exit status 1
I0308 11:06:43.605515       1 run.go:47] Listening for connections on address: &net.UnixAddr{Name:"/csi/chinaunicom.cinder.csi/csi.sock", Net:"unix"}
I0308 11:06:43.955990       1 run.go:68] request funcName = /csi.v1.Identity/GetPluginInfo,request = {}
I0308 11:06:43.956089       1 run.go:83] response funcName = /csi.v1.Identity/GetPluginInfo,response = {"name":"chinaunicom.cinder.csi","vendor_version":"2.0.2"}
I0308 11:06:44.955018       1 run.go:68] request funcName = /csi.v1.Node/NodeGetInfo,request = {}
I0308 11:06:44.958627       1 run.go:83] response funcName = /csi.v1.Node/NodeGetInfo,response = {"node_id":"worker5"}
I0418 16:07:17.411812       1 run.go:68] request funcName = /csi.v1.Node/NodeGetInfo,request = {}
I0418 16:07:17.413391       1 run.go:83] response funcName = /csi.v1.Node/NodeGetInfo,response = {"node_id":"worker5"}
I0423 09:47:59.001905       1 run.go:68] request funcName = /csi.v1.Node/NodeGetInfo,request = {}
I0423 09:47:59.003342       1 run.go:83] response funcName = /csi.v1.Node/NodeGetInfo,response = {"node_id":"worker5"}
```

node-driver-registrar和CSI进程的sock文件：

```
[root@worker1 ~]# tree /var/lib/kubelet/plugins_registry
/var/lib/kubelet/plugins_registry
└── chinaunicom.cinder.csi-reg.sock
[root@worker1 ~]# tree /var/lib/kubelet/plugins
/var/lib/kubelet/plugins
├── chinaunicom.cinder.csi
│   └── csi.sock
├── chinaunicom.cubs.csi
├── cinder.csi.openstack.org
│   └── csi.sock
└── kubernetes.io
    └── csi
        ├── pv
        └── volumeDevices
            ├── publish
            └── staging
```

node对象的annotation：

```sh
[root@master1 ~]# k get no worker4 -oyaml | grep annotations -A 9
  annotations:
    csi.volume.kubernetes.io/nodeid: '{"chinaunicom.cinder.csi":"worker4"}'
    node.alpha.kubernetes.io/ttl: "0"
    projectcalico.org/IPv4Address: 172.31.33.136/24
    projectcalico.org/IPv4IPIPTunnelAddr: 10.6.34.60
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  creationTimestamp: "2022-08-31T01:56:52Z"
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
```

最后验证CSINode对象：

```sh
[root@master1 ~]# k get csinode worker3 -oyaml
apiVersion: storage.k8s.io/v1
kind: CSINode
metadata:
  annotations:
    storage.alpha.kubernetes.io/migrated-plugins: kubernetes.io/aws-ebs,kubernetes.io/azure-disk,kubernetes.io/cinder,kubernetes.io/gce-pd
  creationTimestamp: "2022-08-31T01:56:21Z"
  name: worker3
  ownerReferences:
  - apiVersion: v1
    kind: Node
    name: worker3
    uid: 64a474d6-5b82-4cdc-835a-cb6248283189
  resourceVersion: "2210993225"
  uid: 06c25f8e-b67f-48d2-8c4b-a9c82335248f
spec:
  drivers:
  - name: chinaunicom.lustre.csi
    nodeID: worker3
    topologyKeys: null
  - name: chinaunicom.cinder.csi
    nodeID: worker3
    topologyKeys: null
```
