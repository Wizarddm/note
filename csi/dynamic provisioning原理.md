# Dynamic Provisioning 原理分析

## static provisioning

在kubernetes pod应用中如果有持久化数据的场景，一般会接触到pvc、pv和storageClass这些概念：

- **pvc**: `描述的是pod希望使用的持久化存储的属性`
- **pv**: `描述的是具体的持久化存储数据卷信息`
- **storageClass**: `创建pv的模板`

如果在实际中我们要用到nfs作为pod应用的持久化存储，一般会如下几个步骤：

1. `nfs server端创建一个目录`
2. `创建pv，并配置nfs server和挂载路径`
3. `创建pvc，并与上述pv绑定`
4. `pod引用上述pvc`

暂且抛开第4步，前面3步我们可以通过手动（或者通过程序）一步一步地执行，这种方式在kubernetes中叫作`static provisioning`。相比这种人工管理pv的方式，kubernetes也提供了一套自动创建pv的的机制，叫作`dynamic provisioning`。

## Dynamic Provisioning

所谓的dynamic provisioning，从功能上来讲就是用户基于某个storageClass创建pvc后，会有一个provisioner负责`自动创建卷`、`自动创建pv`并`自动和pvc绑定`。

kubernetes已经支持了一些内置的provisioner，例如AzureDisk、RBD等。如果遇到一些不在kubernetes内置prvisioner范围内的存储（例如nfs）且有自动创建pv需求的场景，就需要我们自己实现一个provisioner。

实现一个provisioner其实很简单，只需要实现如下的`Provisioner`接口即可，其它工作社区维护的库中已经有了相关逻辑：

```go
// sigs.k8s.io/sig-storage-lib-external-provisioner/v8/controller
type Provisioner interface {
    Provision(context.Context, ProvisionOptions) (*v1.PersistentVolume, ProvisioningState, error)
    Delete(context.Context, *v1.PersistentVolume) error
}
```

## dynamic provisoning的不足 

我们知道了dynamic provisioning可以自动创建卷和pv并绑定，但是它的功能也仅仅如此。对于一些复杂的存储，例如谷歌的Persistent Disk，需要`“创建磁盘”`、`挂载到宿主机`、`格式化`、`mount到pod目录`等操作后才能正常使用，很显然这种场景下单纯的provisioning过程是满足不了需求的。面对这些复杂场景，kubernetes抽象出了CSI（Container Storage Interface）来处理。