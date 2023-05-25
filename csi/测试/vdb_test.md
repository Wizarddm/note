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