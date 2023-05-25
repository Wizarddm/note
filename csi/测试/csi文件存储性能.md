# CSI文件存储性能

## 1 测试结论

| 环境             | ops：I/O rate | 带宽：MB sec | 平均响应延时：resp time |
| ---------------- | ------------- | ------------ | ----------------------- |
| 线程1 随机读 4K  | 461.7         | 1.80         | 2.151                   |
| 线程1 随机写 4K  | 1176.8        | 4.60         | 0.837                   |
| 线程1 随机读 1M  | 52.4          | 52.35        | 19.073                  |
| 线程1 随机写 1M  | 57.1          | 57.14        | 17.476                  |
| 线程16 随机读 4K | 1035.3        | 4.04         | 15.436                  |
| 线程16 随机写 4K | 3287.7        | 12.84        | 4.833                   |
| 线程16 随机读 1M | 110.8         | 110.83       | 144.33                  |
| 线程16 随机写 1M | 64.6          | 64.61        | 247.49                  |



runc

| 环境             | ops：I/O rate | 带宽：MB sec | 平均响应延时：resp time |
| ---------------- | ------------- | ------------ | ----------------------- |
| 线程1 随机读 4K  | 3488.0        | 13.63        | 0.098                   |
| 线程1 随机写 4K  | 2939.7        | 11.48        | 0.159                   |
| 线程1 随机读 1M  | 45.1          | 45.05        | 21.933                  |
| 线程1 随机写 1M  | 44.6          | 44.64        | 22.091                  |
| 线程16 随机读 4K | 35835         | 139.98       | 0.233                   |
| 线程16 随机写 4K | 23935         | 93.50        | 0.460                   |
| 线程16 随机读 1M | 302.7         | 302.74       | 52.490                  |
| 线程16 随机写 1M | 240.4         | 240.37       | 66.203                  |



## 2 测试概述



| 线程数 | 类型   | 大小 |
| ------ | ------ | ---- |
| 1      | 随机读 | 4K   |
| 1      | 随机写 | 4K   |
| 1      | 随机读 | 1M   |
| 1      | 随机写 | 1M   |
| 16     | 随机读 | 4K   |
| 16     | 随机写 | 4K   |
| 16     | 随机读 | 1M   |
| 16     | 随机写 | 1M   |

vdbench配置： — Linux下，size=10M 时读性能测试的配置文件test_4K_rw.conf

vdbench运行命令： ./vdbench -f test_4K_rw.conf -o test_4K_log     # test_4K_log为指定结果输出目录

测试结束后，会在最后一行生成测试过程中的平均值：avg_x_x

关注项：

ops：I/O rate：每秒观察到的平均 I/O 速率
带宽：MB sec：传输的数据的平均 MB 数
平均响应延时：resp time：以读/写请求持续时间度量的平均响应时间。所有 vdbench 时间都以毫秒为单位。

线程1 随机读 4K

```
fsd=fsd1,anchor=/mnt/test/dir1,depth=2,width=10,files=1500,size=64K
fwd=fwd1,fsd=fsd1,rdpct=100,xfersize=4K,fileio=random,fileselect=random,threads=1,openflags=directio
rd=rd1,fwd=fwd1,fwdrate=max,format=restart,elapsed=240,interval=1,warmup=120
```

线程1 随机写 4K

```
fsd=fsd1,anchor=/mnt/test/dir1,depth=2,width=10,files=1500,size=64K
fwd=fwd1,fsd=fsd1,rdpct=0,xfersize=4K,fileio=random,fileselect=random,threads=1,openflags=directio
rd=rd1,fwd=fwd1,fwdrate=max,format=restart,elapsed=240,interval=1,warmup=120
```

线程1 随机读 1M

```
fsd=fsd1,anchor=/mnt/test/dir1,depth=2,width=10,files=10,size=10M
fwd=fwd1,fsd=fsd1,rdpct=100,xfersize=1M,fileio=random,fileselect=random,threads=1,openflags=directio
rd=rd1,fwd=fwd1,fwdrate=max,format=restart,elapsed=240,interval=1,warmup=120
```

线程1 随机。写 1M

```
fsd=fsd1,anchor=/mnt/test/dir1,depth=2,width=10,files=10,size=10M
fwd=fwd1,fsd=fsd1,rdpct=0,xfersize=1M,fileio=random,fileselect=random,threads=1,openflags=directio
rd=rd1,fwd=fwd1,fwdrate=max,format=restart,elapsed=240,interval=1,warmup=120
```

线程16 随机读 4K

```
fsd=fsd1,anchor=/mnt/test/dir1,depth=2,width=10,files=1500,size=64K
fwd=fwd1,fsd=fsd1,rdpct=100,xfersize=4K,fileio=random,fileselect=random,threads=16,openflags=directio
rd=rd1,fwd=fwd1,fwdrate=max,format=restart,elapsed=240,interval=1,warmup=120
```

线程16 随机写 4K

```
fsd=fsd1,anchor=/mnt/test/dir1,depth=2,width=10,files=1500,size=64K
fwd=fwd1,fsd=fsd1,rdpct=0,xfersize=4K,fileio=random,fileselect=random,threads=16,openflags=directio
rd=rd1,fwd=fwd1,fwdrate=max,format=restart,elapsed=240,interval=1,warmup=120
```

线程16 随机读 1M

```
fsd=fsd1,anchor=/mnt/test/dir1,depth=2,width=10,files=10,size=10M
fwd=fwd1,fsd=fsd1,rdpct=100,xfersize=1M,fileio=random,fileselect=random,threads=16,openflags=directio
rd=rd1,fwd=fwd1,fwdrate=max,format=restart,elapsed=240,interval=1,warmup=120
```

线程16 随机写 1M

```
fsd=fsd1,anchor=/mnt/test/dir1,depth=2,width=10,files=10,size=10M
fwd=fwd1,fsd=fsd1,rdpct=0,xfersize=1M,fileio=random,fileselect=random,threads=16,openflags=directio
rd=rd1,fwd=fwd1,fwdrate=max,format=restart,elapsed=240,interval=1,warmup=120
```



## 3 kata 测试结果



## 4 runc测试结果

