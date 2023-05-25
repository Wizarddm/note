# kubernetes CSI（上）

# CSI

在介绍CSI之前，先梳理一下kubernetes中使用存储有哪些步骤：

1. 创建pv对象
2. 存储服务器上创建一个volume
3. 把创建的volume挂载到宿主机上
4. 格式化volume
5. 把格式化的volume mount到pod的volume目录
6. 创建pvc对象并和pv对象绑定
7. pod使用pvc

其中1、2、6步是前文[《Dynamic Provisioning原理分析》](https://mp.weixin.qq.com/s?__biz=MzAwMDQyOTcwOA==&mid=2247484763&idx=1&sn=00896bdb313e08c290cd0a3e13a76a8a&scene=21#wechat_redirect)中说到的`Dynamic Provisioning`过程（创建pvc后自动创建pv和volume并绑定），第3步把存储volume挂载到宿主机的过程叫作`Attach`（逆过程叫`Detach`），第4、5两步把volume格式化并mount到pod目录的过程叫作`Mount`（逆过程叫`Unmount`）。

CSI可以理解为kubernetes为了支持`Dynamic Provisioning`、`Attach/Detach`、`Mount/Unmount`等功能的抽象，实现CSI一般需要在`一个进程`里实现三个gRPC service：

- **`IdentityServer`**
- **`ControllerServer`**
- **`NodeServer`**

##### IdentityServer

IdentityServer定义如下：

```go
// github.com/container-storage-interface/spec/lib/go/csi/csi.pb.go
type IdentityServer interface {
    GetPluginInfo(context.Context, *GetPluginInfoRequest) (*GetPluginInfoResponse, error)
    GetPluginCapabilities(context.Context, *GetPluginCapabilitiesRequest) (*GetPluginCapabilitiesResponse, error)
    Probe(context.Context, *ProbeRequest) (*ProbeResponse, error)
}
```

IdentityServer服务负责对外暴露插件本身信息，包括插件名称以及插件所能提供的能力（capabilities）等。

##### ControllerServer

ControllerServer定义如下：

```go
// github.com/container-storage-interface/spec/lib/go/csi/csi.pb.go
type ControllerServer interface {
    CreateVolume(context.Context, *CreateVolumeRequest) (*CreateVolumeResponse, error)
    DeleteVolume(context.Context, *DeleteVolumeRequest) (*DeleteVolumeResponse, error)
    ControllerPublishVolume(context.Context, *ControllerPublishVolumeRequest) (*ControllerPublishVolumeResponse, error)
    ControllerUnpublishVolume(context.Context, *ControllerUnpublishVolumeRequest) (*ControllerUnpublishVolumeResponse, error)
    ValidateVolumeCapabilities(context.Context, *ValidateVolumeCapabilitiesRequest) (*ValidateVolumeCapabilitiesResponse, error)
    ListVolumes(context.Context, *ListVolumesRequest) (*ListVolumesResponse, error)
    GetCapacity(context.Context, *GetCapacityRequest) (*GetCapacityResponse, error)
    ControllerGetCapabilities(context.Context, *ControllerGetCapabilitiesRequest) (*ControllerGetCapabilitiesResponse, error)
    CreateSnapshot(context.Context, *CreateSnapshotRequest) (*CreateSnapshotResponse, error)
    DeleteSnapshot(context.Context, *DeleteSnapshotRequest) (*DeleteSnapshotResponse, error)
    ListSnapshots(context.Context, *ListSnapshotsRequest) (*ListSnapshotsResponse, error)
    ControllerExpandVolume(context.Context, *ControllerExpandVolumeRequest) (*ControllerExpandVolumeResponse, error)
    ControllerGetVolume(context.Context, *ControllerGetVolumeRequest) (*ControllerGetVolumeResponse, error)
}
```

ControllerServer名称中的`controller`可以理解为kubernetes的controller，即通过list&watch某些资源的事件做对应的处理，其实也就是对存储的volume做管理，包括如下等内容：

- `CreateVolume/DeleteVolume`：创建和删除volume，对应的是Dynamic Provisioning过程
- `ControllerPublishVolume/ControllerUnpublishVolume`：对应的是前文提到的Attach/Detach过程
- `CreateSnapshot/DeleteSnapshot`：对应的快照功能
- `ControllerExpandVolume`：对应扩容功能

这里的controller服务是`有状态且部署上与节点无关`的，因此最终的服务部署类型不会是daemonSet，而是带选举的deployment或者副本数为1的statefulSet。

##### NodeServer

NodeServer定义如下：

```go
type NodeServer interface {
    NodeStageVolume(context.Context, *NodeStageVolumeRequest) (*NodeStageVolumeResponse, error)
    NodeUnstageVolume(context.Context, *NodeUnstageVolumeRequest) (*NodeUnstageVolumeResponse, error)
    NodePublishVolume(context.Context, *NodePublishVolumeRequest) (*NodePublishVolumeResponse, error)
    NodeUnpublishVolume(context.Context, *NodeUnpublishVolumeRequest) (*NodeUnpublishVolumeResponse, error)
    NodeGetVolumeStats(context.Context, *NodeGetVolumeStatsRequest) (*NodeGetVolumeStatsResponse, error)
    NodeExpandVolume(context.Context, *NodeExpandVolumeRequest) (*NodeExpandVolumeResponse, error)
    NodeGetCapabilities(context.Context, *NodeGetCapabilitiesRequest) (*NodeGetCapabilitiesResponse, error)
    NodeGetInfo(context.Context, *NodeGetInfoRequest) (*NodeGetInfoResponse, error)
}
```

NodeServer显然就是一些与节点强相关的工作，例如前文提到的Mount，对应的是`NodeStageVolume`和`NodePublishVolume`两个方法，在部署上通常是daemonSet。

# CSI的部署

到这里很多读者可能会很迷惑，因为前文提到CSI需要在一个进程里实现三个gRPC服务，但是同一进程里的三个gRPC服务有的需要deployment/statefulSet部署，有的需要daemonSet部署，这不是一个矛盾话题吗？

在解释这个问题之前我们先了解几个kubernetes社区（https://github.com/kubernetes-csi）维护的项目：

- `node-driver-registrar`：项目地址：https://github.com/kubernetes-csi/node-driver-registrar，主要负责`把CSI注册到kubernetes`。
- `external-provisioner`：项目地址：https://github.com/kubernetes-csi/external-provisioner，主要负责调CSI中`ControllerServer`下的`CreateVolume`和`DeleteVolume`方法实现`Dynamic Provisioning`功能。
- `external-attacher`：项目地址：https://github.com/kubernetes-csi/external-attacher，主要负责调CSI中`ControllerServer`下的`ControllerPublishVolume`和`ControllerUnpublishVolume`方法实现前文提到的`Attach/Detach`操作。
- `external-snapshotter`：项目地址：https://github.com/kubernetes-csi/external-snapshotter，主要负责调CSI中`ControllerServer`下的`CreateSnapshot`和`DeleteSnapshot`实现`快照`功能。
- `external-resizer`：项目地址：https://github.com/kubernetes-csi/external-resizer，主要负责调CSI中`ControllerServer`下的`ControllerExpandVolume`方法实现`扩容`功能。
- `external-health-monitor`：项目地址：https://github.com/kubernetes-csi/external-health-monitor，主要负责`对volume的健康检查`。

这些项目都有官方提供的镜像，是不是都需要和自己写的CSI进程一起部署起来呢？答案是除了`node-driver-registrar`必须选择外，其它的项目都是根据存储特性和业务需求自由选择的。例如nfs没有Attach/Detach过程，我就不需要部署external-attacher服务；再比如我不需要Dynamic Provisioning功能，我就不需要部署external-provisioner服务。

我们再从服务的搭配和部署角度来看看具体应该怎么操作。假设我们现在要部署一个自己开发的nfs CSI，在这个CSI里我需要`Dynamic Provisioning`功能和处理`Mount/Unmount`流程，那么：

1. 首先肯定是要在一个进程里编程实现前面提到的三个gRPC服务对应的方法，例如IdentityServer下的三个方法、ControllerServer下的CreateVolume/DeleteVolume方法以及NodeServer下的NodePublishVolume/NodeUnpublishVolume方法等，并以`unix sock`的方式提供gRPC服务（后文会用到），并打包成镜像；
2. 把`node-driver-registrar`和CSI服务作为两个容器放到一个pod里，这样node-driver-registrar服务就可以用unix sock的方式访问CSI进程里的gRPC服务并且向kubelet注册；
3. node-driver-registrar完成注册后，后续的Mount/Unmount等操作kubelet会直接通过unix sock访问CSI。这里有两层含义：第一层含义是kubelet会直接通过unix sock访问CSI，因此CSI需要用hostPath的方式把自己unix sock文件暴露；第二层含义是kubelet直接调用CSI服务，这意味着`node-driver-registrar`和`CSI`的这个pod应该是daemonSet形式部署的；
4. 把`external-provisioner`和`CSI`服务作为两个容器放到一个pod里，去实现Dynamic Provisioning功能。因为Dynamic Provisioning设计创建卷和删除卷，因此这个pod应该看做是有状态的，在部署上通常是带有选举的deployment部署或者副本数为1的statefulSet部署（如果需要Attach/Detach功能，也可以再加个容器把external-attacher放到这个pod中）。

我们再用个图来总结下整个nfs CSI的逻辑：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3Ulmkt3NWavX4YRkaCcib1NbtUg0PSbVbxyaBD97nnSia0ic2xXEWJRQSj5Yrlj5xK7VFHd4hiayiaVhib14LqdsToew/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

# 总结

本文简单讲述了kubernetes存储的发展历程：in-tree -> flexVolume -> CSI。实现一个CSI需要在一个进程中实现`IdentityServer`、`ControllerServer`和`NodeServer`三个gRPC服务，在部署上通常会有两个负载：一个daemonSet，包含node-driver-registrar和CSI两个服务；一个deployment/statefulSet，包含external-xxx服务和CSI服务（注意这两种负载中都有CSI进程，external-xxx服务和CSI部署在同一个pod里，通常也被叫做`sidecar`）。



# kubernetes CSI（中）

在[《kubernetes CSI（上）》](https://mp.weixin.qq.com/s?__biz=MzAwMDQyOTcwOA==&mid=2247484792&idx=1&sn=d8b18d9b3c8717d94696216e7a464166&scene=21#wechat_redirect)一文中，我们对kubernetes存储的发展有了简单的了解，并且在文中提到了CSI中负责注册的组件`node-driver-registrar`，本文我们基于`github.com/kubernetes-csi/node-driver-registrar@v2.5.1`来分析kubernetes CSI的注册过程。

# 回顾

1. 我们先回顾下一个CSI进程的组成：
![图片](https://mmbiz.qpic.cn/mmbiz_png/3Ulmkt3NWavAZ8gP1S3HFWv9Vibu5JXHvpSVh2LU4vHlgks1pbkzu8bOrpFfdfKojjU9xkiboz0ZjUMiaOCwoj2icg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

一个CSI进程需要实现三个grpc service：`IdentityServer`、`ControllerServer`、`NodeServer`，本文需要关注的是IdentityServer和NodeServer的各一个方法：

```go
// github.com/container-storage-interface/spec/lib/go/csi/csi.pb.go
type IdentityServer interface {
    GetPluginInfo(context.Context, *GetPluginInfoRequest) (*GetPluginInfoResponse, error)
    /*...*/
}

type NodeServer interface {
    NodeGetInfo(context.Context, *NodeGetInfoRequest) (*NodeGetInfoResponse, error)
    /*...*/
}
```

其中：

- GetPluginInfo：返回CSI的信息，主要是名称
- NodeGetInfo：返回一个nodeID，用于后续更新到Node和CSINode对象中
2. 再回顾一下`node-driver-registrar`和`CSI进程`的部署：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3Ulmkt3NWavAZ8gP1S3HFWv9Vibu5JXHvnSeocUBTEjG7FDkQW8XDziaASVmyRIvjbA0pHndrSUOP7iawibfCib0biaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在部署上node-driver-registrar和CSI进程一般以sidecar的形式部署在一个pod里，采用daemonSet部署的原因是因为CSI的注册是通过kubelet完成的，daemonSet可以很好的契合相关场景。

# yaml示例

上面说到node-driver-registrar和CSI进程的部署，我们来看一个yaml部署示例：

```yaml
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: csi-nfs-node
  namespace: kube-system
spec:
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate
  selector:
    matchLabels:
      app: csi-nfs-node
  template:
    metadata:
      labels:
        app: csi-nfs-node
    spec:
      hostNetwork: true  # original nfs connection would be broken without hostNetwork setting
      dnsPolicy: Default  # available values: Default, ClusterFirstWithHostNet, ClusterFirst
      serviceAccountName: csi-nfs-node-sa
      nodeSelector:
        kubernetes.io/os: linux
      tolerations:
        - operator: "Exists"
      containers:
        - name: node-driver-registrar
          image: registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.5.1
          args:
            - --v=2
            - --csi-address=/csi/csi.sock
            - --kubelet-registration-path=$(DRIVER_REG_SOCK_PATH)
          env:
            - name: DRIVER_REG_SOCK_PATH
              value: /var/lib/kubelet/plugins/csi-nfsplugin/csi.sock
            - name: KUBE_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
            - name: registration-dir
              mountPath: /registration
          resources:
            limits:
              memory: 100Mi
            requests:
              cpu: 10m
              memory: 20Mi
        - name: nfs
          securityContext:
            privileged: true
            capabilities:
              add: ["SYS_ADMIN"]
            allowPrivilegeEscalation: true
          image: gcr.io/k8s-staging-sig-storage/nfsplugin:canary
          args:
            - "-v=5"
            - "--nodeid=$(NODE_ID)"
            - "--endpoint=$(CSI_ENDPOINT)"
          env:
            - name: NODE_ID
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: CSI_ENDPOINT
              value: unix:///csi/csi.sock
          ports:
            - containerPort: 29653
              name: healthz
              protocol: TCP
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
            - name: pods-mount-dir
              mountPath: /var/lib/kubelet/pods
              mountPropagation: "Bidirectional"
          resources:
            limits:
              memory: 300Mi
            requests:
              cpu: 10m
              memory: 20Mi
      volumes:
        - name: socket-dir
          hostPath:
            path: /var/lib/kubelet/plugins/csi-nfsplugin
            type: DirectoryOrCreate
        - name: pods-mount-dir
          hostPath:
            path: /var/lib/kubelet/pods
            type: Directory
        - hostPath:
            path: /var/lib/kubelet/plugins_registry
            type: Directory
          name: registration-dir
```

上述示例yaml中有两个容器，一个是node-driver-registrar，另一个是nfs（也就是前文说的的CSI容器），我们主要关注node-driver-registrar容器的`启动参数`和`volumeMounts`配置：

- **启动参数**

1. v：指定容器日志级别
2. `csi-address`：node-driver-registrar需要通过grpc访问CSI进程，这个参数用于指定CSI进程的unix sock文件路径。
3. `kubelet-registration-path`：整个CSI功能是由node-driver-registrar向kubelet注册的，注册完成之后会直接通过unix sock访问CSI进程，因此kubelet-registration-path用于告诉kubelet，CSI进程的sock文件在宿主机上的绝对路径。`kubelet-registration-path参数就是配置CSI进程的sock文件在宿主机上的绝对路径的`。

- **volumeMounts**

1. `socket-dir`：挂载宿主机的/var/lib/kubelet/plugins/csi-nfsplugin目录到容器内的/csi目录，其实就是CSI进程sock所在目录。
2. `registration-dir`：kubelet启动后会基于fsnotify监听宿主机上的/var/lib/kubelet/plugins_registry目录，node-driver-registrar也是以unix sock形式提供grpc服务，因此node-driver-registrar把自己的sock文件按格式放到/var/lib/kubelet/plugins_registry目录下就会触发kubelet的注册过程。registration-dir就是用于把kubelet监听目录挂载到容器内。

# 源码分析

CSI注册过程涉及node-driver-registrar、kubelet和CSI进程，因不同存储的CSI实现会有不同，因此这里主要针对公共逻辑部分代码做分析，即只分析node-driver-registrar和kubelet部分代码。

##### node-driver-registrar源码分析

- **启动参数**

在分析main函数前，我们先看看node-driver-registrar两个非常重要的启动参数：

```go
// node-driver-registrar/cmd/csi-node-driver-registrar/main.go

var (
    csiAddress              = flag.String("csi-address", "/run/csi/socket", "Path of the CSI driver socket that the node-driver-registrar will connect to.")
    kubeletRegistrationPath = flag.String("kubelet-registration-path", "", "Path of the CSI driver socket on the Kubernetes host machine.")
    /*...*/
)
```

和`yaml示例`章节下的启动参数对应，这里再重复解释下：

1. `csi-address`：指定CSI进程的unix sock文件路径。
2. `kubelet-registration-path`：CSI进程的sock文件在宿主机上的绝对路径。

- **main函数**

我们再看看main函数：

```go
// node-driver-registrar/cmd/csi-node-driver-registrar/main.go

func main() {
    /*...*/
    
    csiConn, err := connection.Connect(*csiAddress, cmm)
    
    /*...*/
    
    csiDriverName, err := csirpc.GetDriverName(ctx, csiConn)
    
    /*...*/

    nodeRegister(csiDriverName, addr)
}
```

main函数里逻辑其实非常简单，首先会基于启动参数csi-address初始化一个到CSI进程的连接csiConn，之后基于该连接通过csirpc.GetDriverName拿到csiDriver的名称。

- **GetDriverName**

我们来看看csirpc.GetDriverName的实现：

```go
func GetDriverName(ctx context.Context, conn *grpc.ClientConn) (string, error) {
    client := csi.NewIdentityClient(conn)

    req := csi.GetPluginInfoRequest{}
    rsp, err := client.GetPluginInfo(ctx, &req)
    if err != nil {
        return "", err
    }
    name := rsp.GetName()
    if name == "" {
        return "", fmt.Errorf("driver name is empty")
    }
    return name, nil
}
```

在GetDriverName函数中，会先基于CSI进程的连接初始化一个连接`IdentityServer service`（CSI进程中的一个grpc service，前文有提到），并调用IdentityServer service的`GetPluginInfo`方法，从返回数据里拿到name字段作为csiDriver的名称。

- **nodeRegister**

再看看nodeRegister的实现：

```go
func buildSocketPath(csiDriverName string) string {
    return fmt.Sprintf("%s/%s-reg.sock", *pluginRegistrationPath, csiDriverName)
}

func nodeRegister(csiDriverName, httpEndpoint string) {
    registrar := newRegistrationServer(csiDriverName, *kubeletRegistrationPath, supportedVersions)
    socketPath := buildSocketPath(csiDriverName)
    
    /*...*/
    
    lis, err := net.Listen("unix", socketPath)
    
    /*...*/
    
    grpcServer := grpc.NewServer()
    
    /*...*/

    // Registers kubelet plugin watcher api.
    registerapi.RegisterRegistrationServer(grpcServer, registrar)
    
    /*...*/
    
    // Starts service
    if err := grpcServer.Serve(lis); err != nil {
        klog.Errorf("Registration Server stopped serving: %v", err)
        os.Exit(1)
    }
    
    /*...*/
}
```

结合前文`yaml示例`等章节内容，nodeRegister函数的主要功能就是启动一个rpc服务，并且把该rpc的sock文件放到宿主机的`/var/lib/kubelet/plugins_registry/{csiDriverName}-reg.sock`。这里rpc服务的定义如下：

```go
// k8s.io/kubelet/pkg/apis/pluginregistration/v1/api.pb.go
type RegistrationServer interface {
    GetInfo(context.Context, *InfoRequest) (*PluginInfo, error)
    NotifyRegistrationStatus(context.Context, *RegistrationStatus) (*RegistrationStatusResponse, error)
}
```

RegistrationServer的方法释义如下：

1. `GetInfo`：node-driver-registrar把sock文件放到kubelet监听目录出发kubelet的注册流程后，kubelet会基于该sock文件和rpc方式访问node-driver-registrar的GetInfo方法。`GetInfo方法需要提供csiDriverName、CSI进程sock文件在宿主机上的绝对路径等信息`。
2. `NotifyRegistrationStatus`：`kubelet执行注册逻辑后，会通过rpc回调node-driver-registrar的NotifyRegistrationStatus告知注册结果（成功/失败）`，如果注册失败node-driver-registrar进程会退出，容器会重启，相当于尝试再次注册。

到这里，node-driver-registrar组件的主要逻辑大致分析完了，接下来我们看看kubelet对应的代码逻辑。

##### kubelet相关代码分析

- **fsnotify**

我们先看看kubelet基于fsnotify对目录的监听：

```go
// kubernetes/pkg/kubelet/config/defaults.go
const DefaultKubeletPluginsRegistrationDirName = "plugins_registry"

// kubernetes/pkg/kubelet/kubelet_getters.go
func (kl *Kubelet) getPluginsRegistrationDir() string {
    // kl.getRootDir() ==> /var/lib/kubelet
    return filepath.Join(kl.getRootDir(), config.DefaultKubeletPluginsRegistrationDirName)
}

// kubernetes/pkg/kubelet/kubelet.go
func NewMainKubelet(...){
    /*...*/
    
    klet.pluginManager = pluginmanager.NewPluginManager(
        klet.getPluginsRegistrationDir(), /* sockDir */
        klet.getPluginsDir(),             /* deprecatedSockDir */
        kubeDeps.Recorder,
    )

    /*...*/
}

// kubernetes/pkg/kubelet/pluginmanager/plugin_manager.go
func NewPluginManager(
    sockDir string,
    deprecatedSockDir string,
    recorder record.EventRecorder) PluginManager {
    asw := cache.NewActualStateOfWorld()
    dsw := cache.NewDesiredStateOfWorld()
    reconciler := reconciler.NewReconciler(
        operationexecutor.NewOperationExecutor(
            operationexecutor.NewOperationGenerator(
                recorder,
            ),
        ),
        loopSleepDuration,
        dsw,
        asw,
    )

    pm := &pluginManager{
        desiredStateOfWorldPopulator: pluginwatcher.NewWatcher(
            sockDir,
            deprecatedSockDir,
            dsw,
        ),
        reconciler:          reconciler,
        desiredStateOfWorld: dsw,
        actualStateOfWorld:  asw,
    }
    return pm
}


// kubernetes/pkg/kubelet/pluginmanager/pluginwatcher/plugin_watcher.go
func NewWatcher(sockDir string, deprecatedSockDir string, desiredStateOfWorld cache.DesiredStateOfWorld) *Watcher {
    return &Watcher{
        path:                sockDir,
        deprecatedPath:      deprecatedSockDir,
        fs:                  &utilfs.DefaultFs{},
        desiredStateOfWorld: desiredStateOfWorld,
    }
}

// kubernetes/pkg/kubelet/pluginmanager/pluginwatcher/plugin_watcher.go
func (w *Watcher) Start(stopCh <-chan struct{}) error {
    klog.V(2).Infof("Plugin Watcher Start at %s", w.path)

    w.stopped = make(chan struct{})

    // Creating the directory to be watched if it doesn't exist yet,
    // and walks through the directory to discover the existing plugins.
    if err := w.init(); err != nil {
        return err
    }

    fsWatcher, err := fsnotify.NewWatcher()
    if err != nil {
        return fmt.Errorf("failed to start plugin fsWatcher, err: %v", err)
    }
    w.fsWatcher = fsWatcher

    // Traverse plugin dir and add filesystem watchers before starting the plugin processing goroutine.
    if err := w.traversePluginDir(w.path); err != nil {
        klog.Errorf("failed to traverse plugin socket path %q, err: %v", w.path, err)
    }

    // Traverse deprecated plugin dir, if specified.
    if len(w.deprecatedPath) != 0 {
        if err := w.traversePluginDir(w.deprecatedPath); err != nil {
            klog.Errorf("failed to traverse deprecated plugin socket path %q, err: %v", w.deprecatedPath, err)
        }
    }

    go func(fsWatcher *fsnotify.Watcher) {
        defer close(w.stopped)
        for {
            select {
            case event := <-fsWatcher.Events:
                //TODO: Handle errors by taking corrective measures
                if event.Op&fsnotify.Create == fsnotify.Create {
                    err := w.handleCreateEvent(event)
                    if err != nil {
                        klog.Errorf("error %v when handling create event: %s", err, event)
                    }
                } else if event.Op&fsnotify.Remove == fsnotify.Remove {
                    w.handleDeleteEvent(event)
                }
                continue
            case err := <-fsWatcher.Errors:
                if err != nil {
                    klog.Errorf("fsWatcher received error: %v", err)
                }
                continue
            case <-stopCh:
                // In case of plugin watcher being stopped by plugin manager, stop
                // probing the creation/deletion of plugin sockets.
                // Also give all pending go routines a chance to complete
                select {
                case <-w.stopped:
                case <-time.After(11 * time.Second):
                    klog.Errorf("timeout on stopping watcher")
                }
                w.fsWatcher.Close()
                return
            }
        }
    }(fsWatcher)

    return nil
}
```

上述代码片段描述了kubelet监听/var/lib/kubelet/plugin_registry目录，当该目录下有文件创建或者文件删除的时候，会执行w.handlerCreateEvent/w.handleDeleteEvent函数，这两个函数`仅仅就把node-driver-registrar的sock文件从内存中一个叫desiredStateOfWorld的对象中加入或移除`。难道注册过程仅仅如此？答案是否定的。

- **注册**

其实kubelet在初始化完pluginManager后，会执行pluginManager.Run方法，该方法如下：

```go
// kubernetes/pkg/kubelet/pluginmanager/plugin_manager.go
func (pm *pluginManager) Run(sourcesReady config.SourcesReady, stopCh <-chan struct{}) {
    /*...*/
    
    go pm.reconciler.Run(stopCh)

    /*...*/
}

// kubernetes/pkg/kubelet/pluginmanager/reconciler/reconciler.go
func (rc *reconciler) Run(stopCh <-chan struct{}) {
    wait.Until(func() {
        rc.reconcile()
    },
        rc.loopSleepDuration,
        stopCh)
}

// kubernetes/pkg/kubelet/pluginmanager/reconciler/reconciler.go
func (rc *reconciler) reconcile() {
    // Unregisterations are triggered before registrations

    // Ensure plugins that should be unregistered are unregistered.
    for _, registeredPlugin := range rc.actualStateOfWorld.GetRegisteredPlugins() {
        unregisterPlugin := false
        if !rc.desiredStateOfWorld.PluginExists(registeredPlugin.SocketPath) {
            unregisterPlugin = true
        } else {
            // We also need to unregister the plugins that exist in both actual state of world
            // and desired state of world cache, but the timestamps don't match.
            // Iterate through desired state of world plugins and see if there's any plugin
            // with the same socket path but different timestamp.
            for _, dswPlugin := range rc.desiredStateOfWorld.GetPluginsToRegister() {
                if dswPlugin.SocketPath == registeredPlugin.SocketPath && dswPlugin.Timestamp != registeredPlugin.Timestamp {
                    klog.V(5).Infof(registeredPlugin.GenerateMsgDetailed("An updated version of plugin has been found, unregistering the plugin first before reregistering", ""))
                    unregisterPlugin = true
                    break
                }
            }
        }

        if unregisterPlugin {
            klog.V(5).Infof(registeredPlugin.GenerateMsgDetailed("Starting operationExecutor.UnregisterPlugin", ""))
            err := rc.operationExecutor.UnregisterPlugin(registeredPlugin.SocketPath, rc.getHandlers(), rc.actualStateOfWorld)
            if err != nil &&
                !goroutinemap.IsAlreadyExists(err) &&
                !exponentialbackoff.IsExponentialBackoff(err) {
                // Ignore goroutinemap.IsAlreadyExists and exponentialbackoff.IsExponentialBackoff errors, they are expected.
                // Log all other errors.
                klog.Errorf(registeredPlugin.GenerateErrorDetailed("operationExecutor.UnregisterPlugin failed", err).Error())
            }
            if err == nil {
                klog.V(1).Infof(registeredPlugin.GenerateMsgDetailed("operationExecutor.UnregisterPlugin started", ""))
            }
        }
    }

    // Ensure plugins that should be registered are registered
    for _, pluginToRegister := range rc.desiredStateOfWorld.GetPluginsToRegister() {
        if !rc.actualStateOfWorld.PluginExistsWithCorrectTimestamp(pluginToRegister) {
            klog.V(5).Infof(pluginToRegister.GenerateMsgDetailed("Starting operationExecutor.RegisterPlugin", ""))
            err := rc.operationExecutor.RegisterPlugin(pluginToRegister.SocketPath, pluginToRegister.FoundInDeprecatedDir, pluginToRegister.Timestamp, rc.getHandlers(), rc.actualStateOfWorld)
            if err != nil &&
                !goroutinemap.IsAlreadyExists(err) &&
                !exponentialbackoff.IsExponentialBackoff(err) {
                // Ignore goroutinemap.IsAlreadyExists and exponentialbackoff.IsExponentialBackoff errors, they are expected.
                klog.Errorf(pluginToRegister.GenerateErrorDetailed("operationExecutor.RegisterPlugin failed", err).Error())
            }
            if err == nil {
                klog.V(1).Infof(pluginToRegister.GenerateMsgDetailed("operationExecutor.RegisterPlugin started", ""))
            }
        }
    }
}
```

可以看出，kubelet是专门起了一个协程（goroutine）周期性地处理注册过程，确定某个CSI是否真正的需要向apiServer“注册”是根据actualStateOfWorld和前文提到的desiredStateOfWorld两者数据比较来确定是注册还是移除（取消注册）。如果需要注册，则会执行operationExecutor.RegisterPlugin；如果需要取消注册，则执行operationExecutor.UnregisterPlugin。

先看一下注册的最终实现（部分代码逻辑涉及go语法，这里直接给出关键步骤）：

```go
// kubernetes/pkg/kubelet/pluginmanager/operationexecutor/operation_generator.go
func (og *operationGenerator) GenerateRegisterPluginFunc(
    socketPath string,
    foundInDeprecatedDir bool,
    timestamp time.Time,
    pluginHandlers map[string]cache.PluginHandler,
    actualStateOfWorldUpdater ActualStateOfWorldUpdater) func() error {

    registerPluginFunc := func() error {
        client, conn, err := dial(socketPath, dialTimeoutDuration)
        /*…*/

        infoResp, err := client.GetInfo(ctx, &registerapi.InfoRequest{})
        if err != nil {
            return fmt.Errorf("RegisterPlugin error -- failed to get plugin info using RPC GetInfo at socket %s, err: %v", socketPath, err)
        }

        handler, ok := pluginHandlers[infoResp.Type]
        /*…*/

        if err := handler.RegisterPlugin(infoResp.Name, infoResp.Endpoint, infoResp.SupportedVersions); err != nil {
            return og.notifyPlugin(client, false, fmt.Sprintf("RegisterPlugin error -- plugin registration failed with err: %v", err))
        }

        // Notify is called after register to guarantee that even if notify throws an error Register will always be called after validate
        if err := og.notifyPlugin(client, true, ""); err != nil {
            return fmt.Errorf("RegisterPlugin error -- failed to send registration status at socket %s, err: %v", socketPath, err)
        }
        return nil
    }
    return registerPluginFunc
}

// kubernetes/pkg/kubelet/pluginmanager/operationexecutor/operation_generator.go
func (og *operationGenerator) notifyPlugin(client registerapi.RegistrationClient, registered bool, errStr string) error {
    ctx, cancel := context.WithTimeout(context.Background(), notifyTimeoutDuration)
    defer cancel()

    status := &registerapi.RegistrationStatus{
        PluginRegistered: registered,
        Error:            errStr,
    }

    if _, err := client.NotifyRegistrationStatus(ctx, status); err != nil {
        return errors.Wrap(err, errStr)
    }

    if errStr != "" {
        return errors.New(errStr)
    }

    return nil
}
```

这段代码有两个需要关注的地方：

1. client.GetInfo和client.NotifyRegistrationStatus对应的是node-driver-register grpc的两个方法；
2. handler.RegisterPlugin是真正的注册过程

handler.RegisterPlugin逻辑如下：

```go
// kubernetes/pkg/volume/csi/csi_plugin.go
func (h *RegistrationHandler) RegisterPlugin(pluginName string, endpoint string, versions []string) error {
    /*…*/

    csi, err := newCsiDriverClient(csiDriverName(pluginName))
    /*…*/

    driverNodeID, maxVolumePerNode, accessibleTopology, err := csi.NodeGetInfo(ctx)
    if err != nil {
        if unregErr := unregisterDriver(pluginName); unregErr != nil {
            klog.Error(log("registrationHandler.RegisterPlugin failed to unregister plugin due to previous error: %v", unregErr))
        }
        return err
    }

    err = nim.InstallCSIDriver(pluginName, driverNodeID, maxVolumePerNode, accessibleTopology)
    if err != nil {
        if unregErr := unregisterDriver(pluginName); unregErr != nil {
            klog.Error(log("registrationHandler.RegisterPlugin failed to unregister plugin due to previous error: %v", unregErr))
        }
        return err
    }

    return nil
}

// kubernetes/pkg/volume/csi/csi_plugin.go
func unregisterDriver(driverName string) error {
 csiDrivers.Delete(driverName)

 if err := nim.UninstallCSIDriver(driverName); err != nil {
  return errors.New(log("Error uninstalling CSI driver: %v", err))
 }

 return nil
}

// kubernetes/pkg/volume/csi/nodeinfomanager/nodeinfomanager.go
func (nim *nodeInfoManager) InstallCSIDriver(driverName string, driverNodeID string, maxAttachLimit int64, topology map[string]string) error {
    if driverNodeID == "" {
        return fmt.Errorf("error adding CSI driver node info: driverNodeID must not be empty")
    }

    nodeUpdateFuncs := []nodeUpdateFunc{
        updateNodeIDInNode(driverName, driverNodeID),
    }

    if utilfeature.DefaultFeatureGate.Enabled(features.CSINodeInfo) {
        nodeUpdateFuncs = append(nodeUpdateFuncs, updateTopologyLabels(topology))
    }

    err := nim.updateNode(nodeUpdateFuncs...)
    if err != nil {
        return fmt.Errorf("error updating Node object with CSI driver node info: %v", err)
    }

    if utilfeature.DefaultFeatureGate.Enabled(features.CSINodeInfo) {
        err = nim.updateCSINode(driverName, driverNodeID, maxAttachLimit, topology)
        if err != nil {
            return fmt.Errorf("error updating CSINode object with CSI driver node info: %v", err)
        }
    }
    return nil
}

// kubernetes/pkg/volume/csi/nodeinfomanager/nodeinfomanager.go
func (nim *nodeInfoManager) UninstallCSIDriver(driverName string) error {
    if utilfeature.DefaultFeatureGate.Enabled(features.CSINodeInfo) {
        err := nim.uninstallDriverFromCSINode(driverName)
        if err != nil {
            return fmt.Errorf("error uninstalling CSI driver from CSINode object %v", err)
        }
    }

    err := nim.updateNode(
        removeMaxAttachLimit(driverName),
        removeNodeIDFromNode(driverName),
    )
    if err != nil {
        return fmt.Errorf("error removing CSI driver node info from Node object %v", err)
    }
    return nil
}
```

通过上述代码，我们可以看出kubelet注册其实是做了两个事情：

1. **更新对应node对象的annotation，写入csiDriverName**；
2. **创建或更新CSINode对象**。

# 总结

前面我们从node-driver-registrar和kubelet代码分析了CSI的注册过程，我们可以总结出下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3Ulmkt3NWavAZ8gP1S3HFWv9Vibu5JXHveWvn8dvDKLX26uia67dd4z5r9icWUIAgNEXvGibRqYHZ1NMZLSnjtTQjg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

相关的步骤释义如下：

1. kubelet启动后基于fsnotify监听/var/lib/kubelet/plugin_registry目录；
2. node-driver-registrar启动通过启动参数中配置的CSI进程的sock文件，调CSI进程的`GetPluginInfo`方法获取CSI插件名称；
3. node-driver-registrar启动后在/var/lib/kubelet/plugin_registry目录下创建自己的sock文件`{csiName}-reg.sock`；
4. kubelet的watcher监听到/var/lib/kubelet/plugin_registry目录下有sock文件创建，把该sock文件信息存入内存中的desiredStateOfWorld对象中；
5. kubelet中有个reconciler协程周期性的检查desiredStateOfWorld对象和actualStateOfWorld对象中的数据差异，发现有新的CSI插件需要执行注册过程；
6. reconciler通过/var/lib/kubelet/plugin_registry/{csiName}-reg.sock，调用node-driver-registrar下的`GetInfo`方法获取`CSI插件的名称`和`CSI进程的sock文件路径`等信息；
7. reconciler通过上一步拿到的CSI进程sock文件，调用CSI进程下`NodeGetInfo`方法获取一些数据用于后续的Node和CSINode对象；
8. 组装数据调apiServer接口更新本节点对应的Node对象的annotation；
9. 组装数据调apiServer接口创建/更新对应的CSINode对象；
10. reconciler通过/var/lib/kubelet/plugin_registry/{csiName}-reg.sock，调用node-driver-registrar的`NotifyRegistrationStatus`方法，告知其注册结果。
