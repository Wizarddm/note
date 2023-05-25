## 简介

Dragonfly 是一款基于 P2P 的智能镜像和文件分发工具。它旨在提高大规模文件传输的效率和速率，最大限度地利用网络带宽。在应用分发、缓存分发、日志分发和镜像分发等领域被大规模使用。

## 架构

Dragonfly 架构主要分为三部分 Manager、Scheduler、Seed Peer 以及 Peer 各司其职组成 P2P 下载网络, Dfdaemon 可以作为 Seed Peer 和 Peer。 详细内容可以参考[架构文档](https://d7y.io/zh/docs/concepts/terminology/architecture), 下面是各模块功能:

- **Manager**: 维护各 P2P 集群的关联关系、动态配置管理、用户态以及权限管理等功能。也包含了前端控制台，方便用户进行可视化操作集群。
- **Scheduler**: 为下载节点选择最优下载父节点。异常情况控制 Dfdaemon 回源。
- **Seed Peer**: Dfdaemon 开启 Seed Peer 模式可以作为 P2P 集群中回源下载节点, 也就是整个集群中下载的根节点。
- **Peer**: 通过 Dfdaemon 部署，基于 C/S 架构提供 `dfget` 命令行下载工具，以及 `dfget daemon` 运行守护进程，提供任务下载能力。

![sequence-diagram](https://d7y.io/zh/assets/images/sequence-diagram-36befd084733a4b61f4921ecd76c9568.png)

## 基于 Helm Charts 创建 Dragonfly P2P 集群

创建 Helm Charts 配置文件 `charts-config.yaml`, 配置如下:

```yaml
containerRuntime:
  containerd:
    enable: true
    injectConfigPath: true
    registries:
      - 'https://ghcr.io'

scheduler:
  replicas: 1
  metrics:
    enable: true
  config:
    verbose: true
    pprofPort: 18066

seedPeer:
  replicas: 1
  metrics:
    enable: true
  config:
    verbose: true
    pprofPort: 18066

dfdaemon:
  metrics:
    enable: true
  config:
    verbose: true
    pprofPort: 18066

manager:
  replicas: 1
  metrics:
    enable: true
  config:
    verbose: true
    pprofPort: 18066

jaeger:
  enable: true
```

使用配置文件部署 Dragonfly Helm Charts:

```shell
$ helm repo add dragonfly https://dragonflyoss.github.io/helm-charts/
$ helm install --wait --create-namespace --namespace dragonfly-system dragonfly dragonfly/dragonfly -f charts-config.yaml
NAME: dragonfly
LAST DEPLOYED: Mon Oct 17 18:43:55 2022
NAMESPACE: dragonfly-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the scheduler address by running these commands:
  export SCHEDULER_POD_NAME=$(kubectl get pods --namespace dragonfly-system -l "app=dragonfly,release=dragonfly,component=scheduler" -o jsonpath={.items[0].metadata.name})
  export SCHEDULER_CONTAINER_PORT=$(kubectl get pod --namespace dragonfly-system $SCHEDULER_POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  kubectl --namespace dragonfly-system port-forward $SCHEDULER_POD_NAME 8002:$SCHEDULER_CONTAINER_PORT
  echo "Visit http://127.0.0.1:8002 to use your scheduler"

2. Get the dfdaemon port by running these commands:
  export DFDAEMON_POD_NAME=$(kubectl get pods --namespace dragonfly-system -l "app=dragonfly,release=dragonfly,component=dfdaemon" -o jsonpath={.items[0].metadata.name})
  export DFDAEMON_CONTAINER_PORT=$(kubectl get pod --namespace dragonfly-system $DFDAEMON_POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  You can use $DFDAEMON_CONTAINER_PORT as a proxy port in Node.

3. Configure runtime to use dragonfly:
  https://d7y.io/docs/getting-started/quick-start/kubernetes/


4. Get Jaeger query URL by running these commands:
  export JAEGER_QUERY_PORT=$(kubectl --namespace dragonfly-system get services dragonfly-jaeger-query -o jsonpath="{.spec.ports[0].port}")
  kubectl --namespace dragonfly-system port-forward service/dragonfly-jaeger-query 16686:$JAEGER_QUERY_PORT
  echo "Visit http://127.0.0.1:16686/search?limit=20&lookback=1h&maxDuration&minDuration&service=dragonfly to query download events"
```

检查 Dragonfly 是否部署成功:

```shell
$ kubectl get po -n dragonfly-system
NAME                                READY   STATUS                  RESTARTS      AGE
dragonfly-dfdaemon-lrzmn            0/1     Init:0/3                0             28m
dragonfly-dfdaemon-c956k            1/1     Running                 2 (17h ago)   17h
dragonfly-redis-replicas-1          0/1     Pending                 0             17h
dragonfly-dfdaemon-hj6m7            1/1     Running                 0             17h
dragonfly-dfdaemon-nhw6m            0/1     Init:ImagePullBackOff   0             17h
dragonfly-scheduler-0               1/1     Running                 0             17h
dragonfly-dfdaemon-wmgjg            1/1     Running                 0             17h
dragonfly-dfdaemon-zgs8c            1/1     Running                 0             17h
dragonfly-dfdaemon-4xrc2            1/1     Running                 2 (17h ago)   17h
dragonfly-jaeger-84dbfd5b56-k4pg2   1/1     Running                 0             17h
dragonfly-manager-986cf6b7d-r6k8b   1/1     Running                 0             17h
dragonfly-dfdaemon-46bhc            1/1     Running                 0             17h
dragonfly-dfdaemon-jl9v5            1/1     Running                 0             17h
dragonfly-dfdaemon-mqwmf            0/1     Init:0/3                0             17h
dragonfly-dfdaemon-fnsqh            0/1     Init:0/3                0             17h
dragonfly-redis-replicas-0          1/1     Running                 5 (17h ago)   17h
dragonfly-mysql-0                   1/1     Running                 0             17h
dragonfly-dfdaemon-s6ds5            1/1     Running                 0             17h
dragonfly-seed-peer-0               1/1     Running                 0             17h
dragonfly-dfdaemon-m2ftj            1/1     Running                 0             17h
dragonfly-redis-master-0            1/1     Running                 5 (17h ago)   17h
dragonfly-dfdaemon-p5bhs            1/1     Running                 0             17h
dragonfly-dfdaemon-rwwnc            0/1     Init:0/3                0             17h
dragonfly-dfdaemon-7dxgj            1/1     Running                 1 (17h ago)   17h
dragonfly-dfdaemon-nxgrp            1/1     Running                 0             17h
```

## Containerd 通过 Dragonfly 首次回源拉镜像

在 `kind-worker` Node 下载 `ghcr.io/dragonflyoss/dragonfly2/scheduler:v2.0.5` 镜像:

`worker12`用`docker pull`试一下

```sh
$ ctr -n k8s.io image pull ghcr.io/dragonflyoss/dragonfly2/scheduler:v2.0.5
ghcr.io/dragonflyoss/dragonfly2/scheduler:v2.0.5:                                 resolved       |++++++++++++++++++++++++++++++++++++++| 
manifest-sha256:acf513b53a976859bedf1cf674ccc8766ef8b18e0b60e42bd86a81385796a5a6: done           |++++++++++++++++++++++++++++++++++++++| 
config-sha256:2acfe6ce312b5462f74a5fafab7e21d094431d246715a76d98f6117a369b0ba0:   done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:a8376e5847d0403ffb6fd1277f29f593229945b74b88cfef76dad528cb2bf158:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:82cbeb56bf8065dfb9ff5a0c6ea212ab3a32f413a137675df59d496e68eaf399:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:8a9fba45626f402c12bafaadb718690187cae6e5d56296a8fe7d7c4ce19038f7:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:530afca65e2ea04227630ae746e0c85b2bd1a179379cbf2b6501b49c4cab2ccc:    done           |++++++++++++++++++++++++++++++++++++++| 
elapsed: 181.0s                                                                   total:  26.0 M (147.2 KiB/s)                                     
unpacking linux/amd64 sha256:acf513b53a976859bedf1cf674ccc8766ef8b18e0b60e42bd86a81385796a5a6...
done: 1.755045036s
```

暴露 Jaeger `16686` 端口:

进入Jaeger 页面http://127.0.0.1:16686/search，搜索Tags 值为 `http.url="/v2/dragonflyoss/dragonfly2/scheduler/blobs/sha256:8a9fba45626f402c12bafaadb718690187cae6e5d56296a8fe7d7c4ce19038f7?ns=ghcr.io"` Tracing: