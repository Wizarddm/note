**4K随机读，1 \* 1**

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

**4K随机写，1 \* 1**

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

**4K随机读，8 \* 32**

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

**4K随机写，8 \* 32**

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

**1M随机读，1 \* 1**

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

**1M随机写，1 \* 1**

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

**1M随机读，8 \* 32**

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

**1M随机写，8 \* 32**

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