# `csi` 块存储性能

## 1 **测试结论**

### 1.1 kata 挂载块存储

| 类型             | iops                                                        | bw (  KiB/s)                                                 | lat                                                  |
| ---------------- | ----------------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| 4K随机读，1 * 1  | min=    2, max=  126, avg=73.96, stdev=31.12, samples=533   | min=    8, max=  504, per=100.00%, avg=295.94, stdev=124.47, samples=533 | min=196, max=5539.3k, avg=15205.66, stdev=74290.00   |
| 4K随机写，1 * 1  | min=    2, max=  942, avg=483.25, stdev=223.47, samples=592 | min=    8, max= 3768, per=100.00%, avg=1933.06, stdev=893.89, samples=592 | min=100, max=8810, avg=231.30, stdev=56.51           |
| 4K随机读，8 * 32 | min=    2, max=   94, avg=54.67, stdev= 9.08, samples=4800  | min=    8, max=  376, per=12.51%, avg=218.98, stdev=36.33, samples=4800 | min=18, max=2465, avg=584.71, stdev=276.40           |
| 4K随机写，8 * 32 | min=    1, max=  244, avg=101.95, stdev=60.69, samples=3796 | min=    7, max=  976, per=15.79%, avg=408.05, stdev=242.70, samples=3796 | min=528, max=7903.0k, avg=383582.42, stdev=720306.63 |
| 1M随机读，1 * 1  | min=    2, max=   32, avg=22.92, stdev= 3.71, samples=599   | min= 2048, max=32768, per=100.00%, avg=23497.55, stdev=3794.61, samples=599 | min=11, max=622, avg=43.64, stdev=22.30              |
| 1M随机写，1 * 1  | min=    2, max=   80, avg=36.99, stdev=19.49, samples=552   | min= 2048, max=81920, per=100.00%, avg=37903.58, stdev=19959.79, samples=552 | min=802, max=4659, avg=1411.33, stdev=118.70         |
| 1M随机读，8 * 32 | min=    2, max= 2966, avg=93.11, stdev=312.32, samples=1603 | min= 2048, max=3037184, per=11.52%, avg=95424.32, stdev=319803.61, samples=1603 | min=565, max=8341.3k, avg=316377.83, stdev=674210.57 |
| 1M随机写，8 * 32 | min=    1, max=   26, avg= 5.21, stdev= 2.99, samples=4324  | min= 2043, max=26624, per=13.59%, avg=5518.90, stdev=3055.57, samples=4324 | min=37, max=12513, avg=6229.41, stdev=1231.62        |

### 1.2 runc 挂载块存储

| 类型             | iops                                                         | bw (  KiB/s)                                                 | lat                                                  |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------- |
| 4K随机读，1 * 1  | min=    8, max=  148, avg=102.70, stdev=16.12, samples=600   | min=   32, max=  592, per=100.00%, avg=410.90, stdev=64.48, samples=600 | min=508, max=481839, avg=9727.21, stdev=7188.04      |
| 4K随机写，1 * 1  | min=    2, max= 1564, avg=403.27, stdev=442.83, samples=454  | min=    8, max= 6256, per=100.00%, avg=1613.19, stdev=1771.30, samples=454 | min=389, max=5926.0k, avg=3272.86, stdev=54120.98    |
| 4K随机读，8 * 32 | min=    1, max=  148, avg=59.50, stdev=19.95, samples=4804   | min=    7, max=  592, per=12.53%, avg=238.23, stdev=79.80, samples=4804 | min=342, max=4025.6k, avg=538174.08, stdev=949271.95 |
| 4K随机写，8 * 32 | min=    1, max= 2142, avg=269.63, stdev=338.47, samples=2152 | min=    7, max= 8568, per=28.14%, avg=1078.70, stdev=1353.88, samples=2152 | min=409, max=12789k, avg=267062.76, stdev=1028463.21 |
| 1M随机读，1 * 1  | min=    2, max=   38, avg=26.94, stdev= 4.76, samples=600    | min= 2048, max=38912, per=99.99%, avg=27618.95, stdev=4868.89, samples=600 | min=10, max=549, avg=37.06, stdev=19.57              |
| 1M随机写，1 * 1  | min=    1, max=   94, avg=31.39, stdev=21.22, samples=559    | min= 2043, max=96256, per=100.00%, avg=32186.23, stdev=21732.59, samples=559 | min=10, max=2066, avg=34.12, stdev=91.15             |
| 1M随机读，8 * 32 | min=    1, max=   48, avg=13.39, stdev= 7.41, samples=4563   | min= 1838, max=49152, per=13.30%, avg=13793.41, stdev=7576.98, samples=4563 | min=9, max=22137, avg=2525.21, stdev=5173.97         |
| 1M随机写，8 * 32 | min=    1, max=   82, avg=13.77, stdev=16.63, samples=2822   | min= 2043, max=83968, per=21.35%, avg=14183.77, stdev=17025.89, samples=2822 | min=10, max=20202, avg=3941.77, stdev=3639.94        |



## 2 **测试概述**

### 2.1 **测试场景**

压测容器挂载20G块存储

| 并发主机 | 线程数 | 类型   | 块大小 | 队列深度 | 文件大小 |
| -------- | ------ | ------ | ------ | -------- | -------- |
| 1        | 1      | 随机读 | 4K     | 1        | 10G      |
| 1        | 1      | 随机写 | 4K     | 1        | 10G      |
| 1        | 1      | 随机读 | 1M     | 1        | 10G      |
| 1        | 1      | 随机写 | 1M     | 1        | 10G      |
| 1        | 8      | 随机读 | 4k     | 32       | 10G      |
| 1        | 8      | 随机写 | 4k     | 32       | 10G      |
| 1        | 8      | 随机读 | 1M     | 32       | 10G      |
| 1        | 8      | 随机写 | 1M     | 32       | 10G      |

### 2.2 测试工具

`fio`

### 2.3 测试过程

按照测试场景配置运行文件，执行 fio 配置文件名执行测试：

**4K随机读，1 * 1**

```
[global]
direct=1
group_reporting
ioengine=psync
fsync=1
numjobs=1
runtime=300
filename=/dev/xvd1
name=4k_randread
iodepth=1
rw=randread
bs=4K
size=10G
```

**4K随机写，1 * 1**

```
[global]
direct=1
group_reporting
ioengine=psync
fsync=1
numjobs=1
runtime=300
filename= /dev/xvd1
name=4k_randwrite
iodepth=1
rw=randwrite
bs=4K
size=10G
```

**4K随机读，8 * 32**

```
[global]
direct=1
group_reporting
ioengine=libaio
fsync=1
numjobs=8
runtime=300
filename=/dev/xvd1
name=4k_randread
iodepth=32
rw=randread
bs=4K
size=10G
```

**4K随机写，8 * 32**

```
[global]
direct=1
group_reporting
ioengine=libaio
fsync=1
numjobs=8
runtime=300
filename=/dev/xvd1
name=4k_randread
iodepth=32
rw=randwrite
bs=4K
size=10G
```

**1M随机读，1 * 1**

```
[global]
direct=1
group_reporting
ioengine=psync
fsync=1
numjobs=1
runtime=300
filename=/dev/xvd1
name=1M_randread
iodepth=1
rw=randread
bs=1M
size=10G
```

**1M随机写，1 * 1**

```
[global]
direct=1
group_reporting
ioengine=psync
fsync=1
numjobs=1
runtime=300
filename= /dev/xvd1
name=1M_randwrite
iodepth=1
rw=randwrite
bs=1M
size=10G
```

**1M随机读，8 * 32**

```
[global]
direct=1
group_reporting
ioengine=libaio
fsync=1
numjobs=8
runtime=300
filename=/dev/xvd1
name=1M_randread
iodepth=32
rw=randread
bs=1M
size=10G
```

**1M随机写，8 * 32**

```
[global]
direct=1
group_reporting
ioengine=libaio
fsync=1
numjobs=8
runtime=300
filename=/dev/xvd1
name=1M_randread
iodepth=32
rw=randwrite
bs=1M
size=10G
```

## 3 kata 测试结果

**4K随机读，1 * 1**

```
job: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
fio-3.7
Starting 1 process
Jobs: 1 (f=1): [r(1)][100.0%][r=396KiB/s,w=0KiB/s][r=99,w=0 IOPS][eta 00m:00s]
job: (groupid=0, jobs=1): err= 0: pid=11: Wed Mar 15 11:11:38 2023
   read: IOPS=65, BW=263KiB/s (269kB/s)(77.0MiB/300002msec)
    clat (usec): min=195, max=5539.3k, avg=15204.87, stdev=74290.00
     lat (usec): min=196, max=5539.3k, avg=15205.66, stdev=74290.00
    clat percentiles (msec):
     |  1.00th=[    3],  5.00th=[    5], 10.00th=[    6], 20.00th=[    7],
     | 30.00th=[    8], 40.00th=[    9], 50.00th=[   10], 60.00th=[   11],
     | 70.00th=[   13], 80.00th=[   16], 90.00th=[   21], 95.00th=[   28],
     | 99.00th=[   69], 99.50th=[  140], 99.90th=[  827], 99.95th=[ 1720],
     | 99.99th=[ 2937]
   bw (  KiB/s): min=    8, max=  504, per=100.00%, avg=295.94, stdev=124.47, samples=533
   iops        : min=    2, max=  126, avg=73.96, stdev=31.12, samples=533
  lat (usec)   : 250=0.01%, 1000=0.06%
  lat (msec)   : 2=0.16%, 4=3.94%, 10=46.84%, 20=38.68%, 50=8.89%
  lat (msec)   : 100=0.72%, 250=0.41%, 500=0.13%, 750=0.07%, 1000=0.03%
  cpu          : usr=0.13%, sys=0.30%, ctx=19742, majf=0, minf=12
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=19720,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: bw=263KiB/s (269kB/s), 263KiB/s-263KiB/s (269kB/s-269kB/s), io=77.0MiB (80.8MB), run=300002-300002msec

Disk stats (read/write):
  vda: ios=19708/0, merge=0/0, ticks=298970/0, in_queue=259432, util=100.00%
```

**4K随机写，1 * 1**

```
job: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
fio-3.7
Starting 1 process
Jobs: 1 (f=1): [w(1)][100.0%][r=0KiB/s,w=572KiB/s][r=0,w=143 IOPS][eta 00m:00s]
job: (groupid=0, jobs=1): err= 0: pid=8: Thu Mar 16 09:43:22 2023
  write: IOPS=476, BW=1907KiB/s (1953kB/s)(559MiB/300062msec)
    clat (usec): min=99, max=8809, avg=230.38, stdev=56.49
     lat (usec): min=100, max=8810, avg=231.30, stdev=56.51
    clat percentiles (usec):
     |  1.00th=[  143],  5.00th=[  163], 10.00th=[  176], 20.00th=[  192],
     | 30.00th=[  204], 40.00th=[  217], 50.00th=[  227], 60.00th=[  239],
     | 70.00th=[  255], 80.00th=[  269], 90.00th=[  289], 95.00th=[  297],
     | 99.00th=[  334], 99.50th=[  359], 99.90th=[  424], 99.95th=[  449],
     | 99.99th=[ 1483]
   bw (  KiB/s): min=    8, max= 3768, per=100.00%, avg=1933.06, stdev=893.89, samples=592
   iops        : min=    2, max=  942, avg=483.25, stdev=223.47, samples=592
  lat (usec)   : 100=0.01%, 250=67.10%, 500=32.86%, 750=0.02%, 1000=0.01%
  lat (msec)   : 2=0.01%, 4=0.01%, 10=0.01%
  fsync/fdatasync/sync_file_range:
    sync (usec): min=539, max=1742.5k, avg=1855.45, stdev=14711.67
    sync percentiles (usec):
     |  1.00th=[   660],  5.00th=[   709], 10.00th=[   742], 20.00th=[   783],
     | 30.00th=[   816], 40.00th=[   848], 50.00th=[   873], 60.00th=[   906],
     | 70.00th=[   938], 80.00th=[   988], 90.00th=[  1074], 95.00th=[  1188],
     | 99.00th=[ 14746], 99.50th=[ 40109], 99.90th=[223347], 99.95th=[308282],
     | 99.99th=[541066]
  cpu          : usr=1.55%, sys=2.41%, ctx=286539, majf=0, minf=12
  IO depths    : 1=200.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,143056,0,143056 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=1907KiB/s (1953kB/s), 1907KiB/s-1907KiB/s (1953kB/s-1953kB/s), io=559MiB (586MB), run=300062-300062msec

Disk stats (read/write):
  vda: ios=0/285989, merge=0/0, ticks=0/289640, in_queue=132960, util=100.00%
```

**4K随机读，8 * 32**

```
job: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=32
...
fio-3.7
Starting 8 processes
Jobs: 7 (f=7): [r(6),_(1),r(1)][50.2%][r=404KiB/s,w=0KiB/s][r=101,w=0 IOPS][eta 04m:59s]
job: (groupid=0, jobs=8): err= 0: pid=8: Thu Mar 16 09:51:27 2023
   read: IOPS=437, BW=1751KiB/s (1793kB/s)(515MiB/301048msec)
    slat (usec): min=4, max=237096, avg=29.86, stdev=655.54
    clat (msec): min=18, max=2465, avg=584.68, stdev=276.40
     lat (msec): min=18, max=2465, avg=584.71, stdev=276.40
    clat percentiles (msec):
     |  1.00th=[  355],  5.00th=[  388], 10.00th=[  405], 20.00th=[  426],
     | 30.00th=[  443], 40.00th=[  460], 50.00th=[  477], 60.00th=[  502],
     | 70.00th=[  531], 80.00th=[  600], 90.00th=[ 1133], 95.00th=[ 1250],
     | 99.00th=[ 1418], 99.50th=[ 1502], 99.90th=[ 1703], 99.95th=[ 2366],
     | 99.99th=[ 2433]
   bw (  KiB/s): min=    8, max=  376, per=12.51%, avg=218.98, stdev=36.33, samples=4800
   iops        : min=    2, max=   94, avg=54.67, stdev= 9.08, samples=4800
  lat (msec)   : 20=0.01%, 50=0.01%, 100=0.01%, 250=0.04%, 500=59.97%
  lat (msec)   : 750=24.39%, 1000=1.76%
  cpu          : usr=0.12%, sys=0.26%, ctx=130891, majf=0, minf=333
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=99.8%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=131753,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=1751KiB/s (1793kB/s), 1751KiB/s-1751KiB/s (1793kB/s-1793kB/s), io=515MiB (540MB), run=301048-301048msec

Disk stats (read/write):
  vda: ios=131740/0, merge=0/0, ticks=76872830/0, in_queue=76609288, util=100.00%
```



**4K随机写，8 * 32**

```
job: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=32
...
fio-3.7
Starting 8 processes
Jobs: 8 (f=8): [w(8)][100.0%][r=0KiB/s,w=2496KiB/s][r=0,w=624 IOPS][eta 00m:00s]
job: (groupid=0, jobs=8): err= 0: pid=6: Thu Mar 16 09:59:36 2023
  write: IOPS=646, BW=2586KiB/s (2648kB/s)(758MiB/300003msec)
    slat (usec): min=3, max=1951, avg=22.23, stdev=16.53
    clat (usec): min=372, max=7903.0k, avg=383559.56, stdev=720306.68
     lat (usec): min=528, max=7903.0k, avg=383582.42, stdev=720306.63
    clat percentiles (msec):
     |  1.00th=[  109],  5.00th=[  121], 10.00th=[  128], 20.00th=[  142],
     | 30.00th=[  159], 40.00th=[  180], 50.00th=[  203], 60.00th=[  236],
     | 70.00th=[  305], 80.00th=[  409], 90.00th=[  625], 95.00th=[  902],
     | 99.00th=[ 5000], 99.50th=[ 6141], 99.90th=[ 7886], 99.95th=[ 7886],
     | 99.99th=[ 7886]
   bw (  KiB/s): min=    7, max=  976, per=15.79%, avg=408.05, stdev=242.70, samples=3796
   iops        : min=    1, max=  244, avg=101.95, stdev=60.69, samples=3796
  lat (usec)   : 500=0.01%
  lat (msec)   : 4=0.01%, 10=0.01%, 20=0.01%, 50=0.03%, 100=0.33%
  lat (msec)   : 250=62.84%, 500=21.40%, 750=8.60%, 1000=2.39%
  fsync/fdatasync/sync_file_range:
    sync (nsec): min=228, max=827809, avg=883.65, stdev=2184.07
    sync percentiles (nsec):
     |  1.00th=[  426],  5.00th=[  454], 10.00th=[  474], 20.00th=[  510],
     | 30.00th=[  628], 40.00th=[  908], 50.00th=[  956], 60.00th=[  980],
     | 70.00th=[  996], 80.00th=[ 1032], 90.00th=[ 1096], 95.00th=[ 1176],
     | 99.00th=[ 1336], 99.50th=[ 1464], 99.90th=[18048], 99.95th=[23680],
     | 99.99th=[31872]
  cpu          : usr=0.17%, sys=0.30%, ctx=194337, majf=0, minf=110
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=199.7%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,193948,0,193708 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: bw=2586KiB/s (2648kB/s), 2586KiB/s-2586KiB/s (2648kB/s-2648kB/s), io=758MiB (794MB), run=300003-300003msec

Disk stats (read/write):
  vda: ios=0/387448, merge=0/0, ticks=0/2406714, in_queue=1958784, util=99.98%
```



**1M随机读，1 * 1**

```
job: (g=0): rw=randread, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=psync, iodepth=1
fio-3.7
Starting 1 process
Jobs: 1 (f=1): [r(1)][100.0%][r=20.0MiB/s,w=0KiB/s][r=20,w=0 IOPS][eta 00m:00s]
job: (groupid=0, jobs=1): err= 0: pid=6: Thu Mar 16 10:07:50 2023
   read: IOPS=22, BW=22.9MiB/s (24.0MB/s)(6874MiB/300024msec)
    clat (msec): min=11, max=622, avg=43.64, stdev=22.30
     lat (msec): min=11, max=622, avg=43.64, stdev=22.30
    clat percentiles (msec):
     |  1.00th=[   22],  5.00th=[   24], 10.00th=[   26], 20.00th=[   29],
     | 30.00th=[   32], 40.00th=[   35], 50.00th=[   39], 60.00th=[   44],
     | 70.00th=[   50], 80.00th=[   57], 90.00th=[   67], 95.00th=[   78],
     | 99.00th=[  104], 99.50th=[  118], 99.90th=[  257], 99.95th=[  405],
     | 99.99th=[  625]
   bw (  KiB/s): min= 2048, max=32768, per=100.00%, avg=23497.55, stdev=3794.61, samples=599
   iops        : min=    2, max=   32, avg=22.92, stdev= 3.71, samples=599
  lat (msec)   : 20=0.29%, 50=71.38%, 100=27.19%, 250=1.02%, 500=0.07%
  lat (msec)   : 750=0.04%
  cpu          : usr=0.04%, sys=0.19%, ctx=6912, majf=0, minf=267
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=6874,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: bw=22.9MiB/s (24.0MB/s), 22.9MiB/s-22.9MiB/s (24.0MB/s-24.0MB/s), io=6874MiB (7208MB), run=300024-300024msec

Disk stats (read/write):
  vda: ios=6870/0, merge=0/0, ticks=299292/0, in_queue=285476, util=100.00%
```



**1M随机写，1 * 1**

```
job: (g=0): rw=randwrite, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=psync, iodepth=1
fio-3.7
Starting 1 process
Jobs: 1 (f=1): [w(1)][100.0%][r=0KiB/s,w=54.1MiB/s][r=0,w=54 IOPS][eta 00m:00s]
job: (groupid=0, jobs=1): err= 0: pid=6: Thu Mar 16 10:18:41 2023
  write: IOPS=36, BW=36.8MiB/s (38.6MB/s)(10.0GiB/278281msec)
    clat (usec): min=778, max=4594, avg=1343.60, stdev=106.61
     lat (usec): min=802, max=4659, avg=1411.33, stdev=118.70
    clat percentiles (usec):
     |  1.00th=[ 1156],  5.00th=[ 1205], 10.00th=[ 1237], 20.00th=[ 1270],
     | 30.00th=[ 1303], 40.00th=[ 1319], 50.00th=[ 1336], 60.00th=[ 1352],
     | 70.00th=[ 1385], 80.00th=[ 1418], 90.00th=[ 1450], 95.00th=[ 1483],
     | 99.00th=[ 1565], 99.50th=[ 1680], 99.90th=[ 2376], 99.95th=[ 2802],
     | 99.99th=[ 3490]
   bw (  KiB/s): min= 2048, max=81920, per=100.00%, avg=37903.58, stdev=19959.79, samples=552
   iops        : min=    2, max=   80, avg=36.99, stdev=19.49, samples=552
  lat (usec)   : 1000=0.15%
  lat (msec)   : 2=99.72%, 4=0.13%, 10=0.01%
  fsync/fdatasync/sync_file_range:
    sync (msec): min=10, max=1010, avg=25.76, stdev=49.03
    sync percentiles (msec):
     |  1.00th=[   11],  5.00th=[   11], 10.00th=[   12], 20.00th=[   12],
     | 30.00th=[   12], 40.00th=[   12], 50.00th=[   12], 60.00th=[   12],
     | 70.00th=[   12], 80.00th=[   16], 90.00th=[   54], 95.00th=[  102],
     | 99.00th=[  247], 99.50th=[  338], 99.90th=[  518], 99.95th=[  634],
     | 99.99th=[  684]
  cpu          : usr=0.35%, sys=0.30%, ctx=20523, majf=0, minf=11
  IO depths    : 1=200.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,10240,0,10239 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=36.8MiB/s (38.6MB/s), 36.8MiB/s-36.8MiB/s (38.6MB/s-38.6MB/s), io=10.0GiB (10.7GB), run=278281-278281msec

Disk stats (read/write):
  vda: ios=0/20455, merge=0/0, ticks=0/276482, in_queue=233788, util=100.00%
```



**1M随机读，8 * 32**

```
job: (g=0): rw=randread, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
fio-3.7
Starting 8 processes
Jobs: 8 (f=8): [r(8)][86.3%][r=21.4GiB/s,w=0KiB/s][r=21.9k,w=0 IOPS][eta 00m:16s]
job: (groupid=0, jobs=8): err= 0: pid=10: Thu Mar 16 10:23:43 2023
   read: IOPS=808, BW=809MiB/s (848MB/s)(80.0GiB/101312msec)
    slat (usec): min=15, max=22923, avg=76.03, stdev=281.61
    clat (usec): min=530, max=8341.3k, avg=316301.11, stdev=674202.99
     lat (usec): min=565, max=8341.3k, avg=316377.83, stdev=674210.57
    clat percentiles (msec):
     |  1.00th=[    4],  5.00th=[    6], 10.00th=[    8], 20.00th=[   11],
     | 30.00th=[   14], 40.00th=[   18], 50.00th=[   31], 60.00th=[   69],
     | 70.00th=[  167], 80.00th=[  435], 90.00th=[ 1045], 95.00th=[ 1586],
     | 99.00th=[ 3742], 99.50th=[ 4396], 99.90th=[ 5067], 99.95th=[ 5403],
     | 99.99th=[ 6409]
   bw (  KiB/s): min= 2048, max=3037184, per=11.52%, avg=95424.32, stdev=319803.61, samples=1603
   iops        : min=    2, max= 2966, avg=93.11, stdev=312.32, samples=1603
  lat (usec)   : 750=0.01%, 1000=0.02%
  lat (msec)   : 2=0.24%, 4=1.57%, 10=17.24%, 20=24.39%, 50=12.92%
  lat (msec)   : 100=8.11%, 250=9.76%, 500=7.66%, 750=4.75%, 1000=2.83%
  cpu          : usr=0.14%, sys=0.82%, ctx=52851, majf=0, minf=65632
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.2%, 32=99.7%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=81920,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=809MiB/s (848MB/s), 809MiB/s-809MiB/s (848MB/s-848MB/s), io=80.0GiB (85.9GB), run=101312-101312msec

Disk stats (read/write):
  vda: ios=77898/0, merge=0/0, ticks=25817629/0, in_queue=25661956, util=100.00%
```



**1M随机写，8 * 32**

```
job: (g=0): rw=randwrite, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
fio-3.7
Starting 8 processes
Jobs: 8 (f=8): [w(8)][100.0%][r=0KiB/s,w=39.0MiB/s][r=0,w=39 IOPS][eta 00m:00s]
job: (groupid=0, jobs=8): err= 0: pid=6: Thu Mar 16 10:31:55 2023
  write: IOPS=39, BW=39.6MiB/s (41.6MB/s)(11.6GiB/300289msec)
    slat (usec): min=36, max=31616, avg=176.88, stdev=352.98
    clat (msec): min=37, max=12513, avg=6229.24, stdev=1231.62
     lat (msec): min=37, max=12513, avg=6229.41, stdev=1231.62
    clat percentiles (msec):
     |  1.00th=[ 2937],  5.00th=[ 4245], 10.00th=[ 4732], 20.00th=[ 5470],
     | 30.00th=[ 5738], 40.00th=[ 6007], 50.00th=[ 6208], 60.00th=[ 6477],
     | 70.00th=[ 6745], 80.00th=[ 7080], 90.00th=[ 7617], 95.00th=[ 8087],
     | 99.00th=[ 9463], 99.50th=[10402], 99.90th=[12281], 99.95th=[12416],
     | 99.99th=[12550]
   bw (  KiB/s): min= 2043, max=26624, per=13.59%, avg=5518.90, stdev=3055.57, samples=4324
   iops        : min=    1, max=   26, avg= 5.21, stdev= 2.99, samples=4324
  lat (msec)   : 50=0.02%, 100=0.02%, 500=0.13%, 750=0.03%, 1000=0.05%
  fsync/fdatasync/sync_file_range:
    sync (nsec): min=281, max=50219, avg=1071.98, stdev=1499.63
    sync percentiles (nsec):
     |  1.00th=[  490],  5.00th=[  532], 10.00th=[  564], 20.00th=[  660],
     | 30.00th=[  804], 40.00th=[  916], 50.00th=[  956], 60.00th=[ 1012],
     | 70.00th=[ 1096], 80.00th=[ 1176], 90.00th=[ 1320], 95.00th=[ 1560],
     | 99.00th=[ 2448], 99.50th=[12864], 99.90th=[25472], 99.95th=[30848],
     | 99.99th=[39680]
  cpu          : usr=0.08%, sys=0.03%, ctx=11859, majf=0, minf=103
  IO depths    : 1=0.1%, 2=0.2%, 4=0.3%, 8=0.5%, 16=1.1%, 32=195.8%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=99.9%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,11905,0,11665 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: bw=39.6MiB/s (41.6MB/s), 39.6MiB/s-39.6MiB/s (41.6MB/s-41.6MB/s), io=11.6GiB (12.5GB), run=300289-300289msec

Disk stats (read/write):
  vda: ios=0/23569, merge=0/0, ticks=0/2437746, in_queue=2389212, util=100.00%
```

## 4 runc 测试结果

4K随机读，1 * 1

```
job: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
fio-3.7
Starting 1 process
Jobs: 1 (f=1): [r(1)][100.0%][r=364KiB/s,w=0KiB/s][r=91,w=0 IOPS][eta 00m:00s]
job: (groupid=0, jobs=1): err= 0: pid=56: Thu Mar 16 11:07:42 2023
   read: IOPS=102, BW=411KiB/s (421kB/s)(120MiB/300005msec)
    clat (usec): min=507, max=481839, avg=9726.57, stdev=7188.04
     lat (usec): min=508, max=481839, avg=9727.21, stdev=7188.04
    clat percentiles (msec):
     |  1.00th=[    3],  5.00th=[    4], 10.00th=[    5], 20.00th=[    6],
     | 30.00th=[    7], 40.00th=[    8], 50.00th=[    9], 60.00th=[   10],
     | 70.00th=[   11], 80.00th=[   13], 90.00th=[   17], 95.00th=[   22],
     | 99.00th=[   34], 99.50th=[   39], 99.90th=[   63], 99.95th=[   78],
     | 99.99th=[  124]
   bw (  KiB/s): min=   32, max=  592, per=100.00%, avg=410.90, stdev=64.48, samples=600
   iops        : min=    8, max=  148, avg=102.70, stdev=16.12, samples=600
  lat (usec)   : 750=0.14%, 1000=0.05%
  lat (msec)   : 2=0.30%, 4=9.40%, 10=55.22%, 20=28.98%, 50=5.73%
  lat (msec)   : 100=0.15%, 250=0.02%, 500=0.01%
  cpu          : usr=0.11%, sys=0.27%, ctx=61297, majf=0, minf=10
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=30823,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: bw=411KiB/s (421kB/s), 411KiB/s-411KiB/s (421kB/s-421kB/s), io=120MiB (126MB), run=300005-300005msec

Disk stats (read/write):
  rbd0: ios=30810/0, merge=0/0, ticks=297843/0, in_queue=297843, util=100.00%
```



4K随机写，1 * 1

```
job: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
fio-3.7
Starting 1 process
Jobs: 1 (f=1): [w(1)][100.0%][r=0KiB/s,w=180KiB/s][r=0,w=45 IOPS][eta 00m:00s]  
job: (groupid=0, jobs=1): err= 0: pid=56: Thu Mar 16 11:14:48 2023
  write: IOPS=304, BW=1219KiB/s (1248kB/s)(358MiB/300560msec)
    clat (usec): min=388, max=5926.0k, avg=3272.62, stdev=54120.98
     lat (usec): min=389, max=5926.0k, avg=3272.86, stdev=54120.98
    clat percentiles (usec):
     |  1.00th=[    445],  5.00th=[    469], 10.00th=[    486],
     | 20.00th=[    510], 30.00th=[    529], 40.00th=[    545],
     | 50.00th=[    570], 60.00th=[    594], 70.00th=[    635],
     | 80.00th=[    676], 90.00th=[    775], 95.00th=[   1020],
     | 99.00th=[  23725], 99.50th=[ 105382], 99.90th=[ 541066],
     | 99.95th=[1019216], 99.99th=[2264925]
   bw (  KiB/s): min=    8, max= 6256, per=100.00%, avg=1613.19, stdev=1771.30, samples=454
   iops        : min=    2, max= 1564, avg=403.27, stdev=442.83, samples=454
  lat (usec)   : 500=16.01%, 750=72.39%, 1000=6.48%
  lat (msec)   : 2=2.52%, 4=0.67%, 10=0.47%, 20=0.39%, 50=0.30%
  lat (msec)   : 100=0.25%, 250=0.26%, 500=0.15%, 750=0.03%, 1000=0.02%
  fsync/fdatasync/sync_file_range:
    sync (nsec): min=1820, max=47254, avg=4253.39, stdev=1085.49
    sync percentiles (nsec):
     |  1.00th=[ 3248],  5.00th=[ 3472], 10.00th=[ 3664], 20.00th=[ 3856],
     | 30.00th=[ 3920], 40.00th=[ 3984], 50.00th=[ 4048], 60.00th=[ 4128],
     | 70.00th=[ 4192], 80.00th=[ 4384], 90.00th=[ 5280], 95.00th=[ 5472],
     | 99.00th=[ 6240], 99.50th=[ 9792], 99.90th=[19072], 99.95th=[21888],
     | 99.99th=[26240]
  cpu          : usr=0.32%, sys=0.66%, ctx=91599, majf=0, minf=174
  IO depths    : 1=200.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,91563,0,91562 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=1219KiB/s (1248kB/s), 1219KiB/s-1219KiB/s (1248kB/s-1248kB/s), io=358MiB (375MB), run=300560-300560msec

Disk stats (read/write):
  rbd0: ios=0/91562, merge=0/0, ticks=0/295277, in_queue=295277, util=100.00%
```



4K随机读，8 * 32

```
job: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=32
...
fio-3.7
Starting 8 processes
Jobs: 5 (f=0): [f(2),_(1),f(2),_(2),f(1)][100.0%][r=992KiB/s,w=0KiB/s][r=248,w=0 IOPS][eta 00m:00s]
job: (groupid=0, jobs=8): err= 0: pid=56: Thu Mar 16 11:24:02 2023
   read: IOPS=475, BW=1902KiB/s (1948kB/s)(560MiB/301499msec)
    slat (usec): min=4, max=1231, avg=107.69, stdev=24.72
    clat (usec): min=207, max=4025.5k, avg=538065.49, stdev=949271.94
     lat (usec): min=342, max=4025.6k, avg=538174.08, stdev=949271.95
    clat percentiles (msec):
     |  1.00th=[    4],  5.00th=[    8], 10.00th=[   12], 20.00th=[   21],
     | 30.00th=[   33], 40.00th=[   52], 50.00th=[   82], 60.00th=[  138],
     | 70.00th=[  262], 80.00th=[  651], 90.00th=[ 2534], 95.00th=[ 2869],
     | 99.00th=[ 3406], 99.50th=[ 3574], 99.90th=[ 3876], 99.95th=[ 3943],
     | 99.99th=[ 3977]
   bw (  KiB/s): min=    7, max=  592, per=12.53%, avg=238.23, stdev=79.80, samples=4804
   iops        : min=    1, max=  148, avg=59.50, stdev=19.95, samples=4804
  lat (usec)   : 250=0.01%, 500=0.41%, 750=0.09%, 1000=0.02%
  lat (msec)   : 2=0.05%, 4=0.62%, 10=6.91%, 20=11.82%, 50=19.39%
  lat (msec)   : 100=14.92%, 250=15.17%, 500=8.24%, 750=3.44%, 1000=1.91%
  cpu          : usr=0.08%, sys=0.20%, ctx=279113, majf=0, minf=355
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=99.8%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=143357,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=1902KiB/s (1948kB/s), 1902KiB/s-1902KiB/s (1948kB/s-1948kB/s), io=560MiB (587MB), run=301499-301499msec

Disk stats (read/write):
  rbd0: ios=143347/0, merge=0/0, ticks=76924879/0, in_queue=76924879, util=100.00%
```



4K随机写，8 * 32

```
job: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=32
...
fio-3.7
Starting 8 processes
Jobs: 8 (f=8): [w(8)][1.4%][r=0KiB/s,w=0KiB/s][r=0,w=0 IOPS][eta 06h:02m:23s]    
job: (groupid=0, jobs=8): err= 0: pid=56: Thu Mar 16 11:31:03 2023
  write: IOPS=958, BW=3834KiB/s (3926kB/s)(1135MiB/303122msec)
    slat (usec): min=2, max=6030, avg=114.38, stdev=93.88
    clat (usec): min=60, max=12789k, avg=266947.74, stdev=1028460.43
     lat (usec): min=409, max=12789k, avg=267062.76, stdev=1028463.21
    clat percentiles (usec):
     |  1.00th=[     502],  5.00th=[     758], 10.00th=[    1123],
     | 20.00th=[    2245], 30.00th=[    4080], 40.00th=[    7767],
     | 50.00th=[   13960], 60.00th=[   24511], 70.00th=[   51119],
     | 80.00th=[  143655], 90.00th=[  425722], 95.00th=[  935330],
     | 99.00th=[ 6274679], 99.50th=[ 6811550], 99.90th=[ 8556381],
     | 99.95th=[12683576], 99.99th=[12817794]
   bw (  KiB/s): min=    7, max= 8568, per=28.14%, avg=1078.70, stdev=1353.88, samples=2152
   iops        : min=    1, max= 2142, avg=269.63, stdev=338.47, samples=2152
  lat (usec)   : 100=0.01%, 250=0.01%, 500=0.97%, 750=3.95%, 1000=3.62%
  lat (msec)   : 2=9.78%, 4=11.33%, 10=14.39%, 20=12.56%, 50=13.13%
  lat (msec)   : 100=7.17%, 250=8.22%, 500=5.93%, 750=2.86%, 1000=1.39%
  fsync/fdatasync/sync_file_range:
    sync (nsec): min=54, max=45871, avg=244.32, stdev=380.86
    sync percentiles (nsec):
     |  1.00th=[  102],  5.00th=[  117], 10.00th=[  127], 20.00th=[  147],
     | 30.00th=[  171], 40.00th=[  201], 50.00th=[  237], 60.00th=[  270],
     | 70.00th=[  286], 80.00th=[  314], 90.00th=[  358], 95.00th=[  386],
     | 99.00th=[  454], 99.50th=[  498], 99.90th=[  612], 99.95th=[ 2096],
     | 99.99th=[18304]
  cpu          : usr=0.13%, sys=0.30%, ctx=438050, majf=0, minf=401
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=199.8%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,290530,0,290282 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: bw=3834KiB/s (3926kB/s), 3834KiB/s-3834KiB/s (3926kB/s-3926kB/s), io=1135MiB (1190MB), run=303122-303122msec

Disk stats (read/write):
  rbd0: ios=0/290274, merge=0/0, ticks=0/76094657, in_queue=76094658, util=100.00%
```



1M随机读，1 * 1

```
job: (g=0): rw=randread, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=psync, iodepth=1
fio-3.7
Starting 1 process
Jobs: 1 (f=1): [r(1)][100.0%][r=27.0MiB/s,w=0KiB/s][r=27,w=0 IOPS][eta 00m:00s]
job: (groupid=0, jobs=1): err= 0: pid=56: Thu Mar 16 14:40:14 2023
   read: IOPS=26, BW=26.0MiB/s (28.3MB/s)(8093MiB/300007msec)
    clat (msec): min=10, max=549, avg=37.06, stdev=19.57
     lat (msec): min=10, max=549, avg=37.06, stdev=19.57
    clat percentiles (msec):
     |  1.00th=[   12],  5.00th=[   21], 10.00th=[   23], 20.00th=[   26],
     | 30.00th=[   29], 40.00th=[   32], 50.00th=[   34], 60.00th=[   38],
     | 70.00th=[   41], 80.00th=[   46], 90.00th=[   54], 95.00th=[   62],
     | 99.00th=[   85], 99.50th=[  114], 99.90th=[  255], 99.95th=[  397],
     | 99.99th=[  550]
   bw (  KiB/s): min= 2048, max=38912, per=99.99%, avg=27618.95, stdev=4868.89, samples=600
   iops        : min=    2, max=   38, avg=26.94, stdev= 4.76, samples=600
  lat (msec)   : 20=4.30%, 50=82.43%, 100=12.65%, 250=0.49%, 500=0.11%
  lat (msec)   : 750=0.01%
  cpu          : usr=0.03%, sys=0.24%, ctx=15603, majf=0, minf=265
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=8093,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: bw=26.0MiB/s (28.3MB/s), 26.0MiB/s-26.0MiB/s (28.3MB/s-28.3MB/s), io=8093MiB (8486MB), run=300007-300007msec

Disk stats (read/write):
  rbd0: ios=8088/0, merge=0/0, ticks=298826/0, in_queue=298825, util=100.00%
sh-4.2# 
```



1M随机写，1 * 1

```
job: (g=0): rw=randwrite, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=psync, iodepth=1
fio-3.7
Starting 1 process
Jobs: 1 (f=1): [w(1)][100.0%][r=0KiB/s,w=0KiB/s][r=0,w=0 IOPS][eta 00m:00s]   
job: (groupid=0, jobs=1): err= 0: pid=57: Thu Mar 16 14:47:14 2023
  write: IOPS=29, BW=29.3MiB/s (30.7MB/s)(8791MiB/300032msec)
    clat (msec): min=9, max=2066, avg=34.08, stdev=91.15
     lat (msec): min=10, max=2066, avg=34.12, stdev=91.15
    clat percentiles (msec):
     |  1.00th=[   11],  5.00th=[   11], 10.00th=[   11], 20.00th=[   11],
     | 30.00th=[   11], 40.00th=[   11], 50.00th=[   11], 60.00th=[   11],
     | 70.00th=[   12], 80.00th=[   21], 90.00th=[   68], 95.00th=[  142],
     | 99.00th=[  439], 99.50th=[  584], 99.90th=[ 1200], 99.95th=[ 1284],
     | 99.99th=[ 2072]
   bw (  KiB/s): min= 2043, max=96256, per=100.00%, avg=32186.23, stdev=21732.59, samples=559
   iops        : min=    1, max=   94, avg=31.39, stdev=21.22, samples=559
  lat (msec)   : 10=0.02%, 20=79.77%, 50=8.00%, 100=4.98%, 250=4.82%
  lat (msec)   : 500=1.66%, 750=0.39%, 1000=0.19%
  fsync/fdatasync/sync_file_range:
    sync (nsec): min=2039, max=35965, avg=5479.64, stdev=1526.88
    sync percentiles (nsec):
     |  1.00th=[ 3856],  5.00th=[ 4512], 10.00th=[ 4640], 20.00th=[ 5088],
     | 30.00th=[ 5280], 40.00th=[ 5344], 50.00th=[ 5408], 60.00th=[ 5472],
     | 70.00th=[ 5536], 80.00th=[ 5536], 90.00th=[ 5600], 95.00th=[ 6112],
     | 99.00th=[ 9920], 99.50th=[17280], 99.90th=[25472], 99.95th=[26240],
     | 99.99th=[36096]
  cpu          : usr=0.14%, sys=0.21%, ctx=8815, majf=0, minf=275
  IO depths    : 1=200.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,8791,0,8790 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=29.3MiB/s (30.7MB/s), 29.3MiB/s-29.3MiB/s (30.7MB/s-30.7MB/s), io=8791MiB (9218MB), run=300032-300032msec

Disk stats (read/write):
  rbd0: ios=0/8787, merge=0/0, ticks=0/297406, in_queue=297406, util=100.00%
```



1M随机读，8 * 32

```
job: (g=0): rw=randread, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
fio-3.7
Starting 8 processes
Jobs: 8 (f=0): [f(8)][100.0%][r=219MiB/s,w=0KiB/s][r=219,w=0 IOPS][eta 00m:00s]
job: (groupid=0, jobs=8): err= 0: pid=56: Thu Mar 16 14:55:00 2023
   read: IOPS=101, BW=101MiB/s (106MB/s)(30.3GiB/306115msec)
    slat (usec): min=14, max=1923, avg=117.75, stdev=73.68
    clat (msec): min=9, max=22137, avg=2525.09, stdev=5173.97
     lat (msec): min=9, max=22137, avg=2525.21, stdev=5173.97
    clat percentiles (msec):
     |  1.00th=[   18],  5.00th=[   31], 10.00th=[   39], 20.00th=[   57],
     | 30.00th=[   82], 40.00th=[  117], 50.00th=[  176], 60.00th=[  292],
     | 70.00th=[  523], 80.00th=[ 1301], 90.00th=[14160], 95.00th=[15503],
     | 99.00th=[17113], 99.50th=[17113], 99.90th=[17113], 99.95th=[17113],
     | 99.99th=[17113]
   bw (  KiB/s): min= 1838, max=49152, per=13.30%, avg=13793.41, stdev=7576.98, samples=4563
   iops        : min=    1, max=   48, avg=13.39, stdev= 7.41, samples=4563
  lat (msec)   : 10=0.36%, 20=1.01%, 50=14.65%, 100=19.92%, 250=21.25%
  lat (msec)   : 500=12.04%, 750=5.57%, 1000=3.39%
  cpu          : usr=0.02%, sys=0.07%, ctx=55746, majf=0, minf=4306
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.2%, 16=0.4%, 32=99.2%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=31002,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=101MiB/s (106MB/s), 101MiB/s-101MiB/s (106MB/s-106MB/s), io=30.3GiB (32.5GB), run=306115-306115msec

Disk stats (read/write):
  rbd0: ios=30991/0, merge=0/0, ticks=77488756/0, in_queue=77488756, util=100.00%
```



1M随机写，8 * 32

```
job: (g=0): rw=randwrite, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
fio-3.7
Starting 8 processes
Jobs: 8 (f=8): [w(8)][23.3%][r=0KiB/s,w=0KiB/s][r=0,w=0 IOPS][eta 16m:44s]     
job: (groupid=0, jobs=8): err= 0: pid=56: Thu Mar 16 15:06:59 2023
  write: IOPS=64, BW=64.9MiB/s (68.0MB/s)(19.3GiB/305079msec)
    slat (usec): min=40, max=45807, avg=385.84, stdev=873.48
    clat (msec): min=9, max=20202, avg=3941.38, stdev=3640.04
     lat (msec): min=10, max=20202, avg=3941.77, stdev=3639.94
    clat percentiles (msec):
     |  1.00th=[   36],  5.00th=[  138], 10.00th=[  292], 20.00th=[  617],
     | 30.00th=[ 1011], 40.00th=[ 1569], 50.00th=[ 2702], 60.00th=[ 4933],
     | 70.00th=[ 6141], 80.00th=[ 6275], 90.00th=[ 9194], 95.00th=[11342],
     | 99.00th=[14697], 99.50th=[15905], 99.90th=[17113], 99.95th=[17113],
     | 99.99th=[17113]
   bw (  KiB/s): min= 2043, max=83968, per=21.35%, avg=14183.77, stdev=17025.89, samples=2822
   iops        : min=    1, max=   82, avg=13.77, stdev=16.63, samples=2822
  lat (msec)   : 10=0.04%, 20=0.39%, 50=1.13%, 100=2.13%, 250=4.99%
  lat (msec)   : 500=8.03%, 750=6.88%, 1000=5.98%
  fsync/fdatasync/sync_file_range:
    sync (nsec): min=59, max=26317, avg=313.46, stdev=509.20
    sync percentiles (nsec):
     |  1.00th=[  121],  5.00th=[  163], 10.00th=[  201], 20.00th=[  255],
     | 30.00th=[  266], 40.00th=[  274], 50.00th=[  282], 60.00th=[  302],
     | 70.00th=[  334], 80.00th=[  358], 90.00th=[  398], 95.00th=[  430],
     | 99.00th=[  604], 99.50th=[  772], 99.90th=[ 1576], 99.95th=[17280],
     | 99.99th=[22656]
  cpu          : usr=0.10%, sys=0.04%, ctx=36885, majf=0, minf=101
  IO depths    : 1=0.1%, 2=0.1%, 4=0.2%, 8=0.3%, 16=0.6%, 32=197.5%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,19796,0,19548 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: bw=64.9MiB/s (68.0MB/s), 64.9MiB/s-64.9MiB/s (68.0MB/s-68.0MB/s), io=19.3GiB (20.8GB), run=305079-305079msec

Disk stats (read/write):
  rbd0: ios=0/19701, merge=0/0, ticks=0/76214525, in_queue=76214524, util=100.00%
```

