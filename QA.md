### Q: 执行脚本，发现报错：**-bash: ./shell1: /bin/bash^M: 解释器错误: 没有那个文件或目录**

**A**:  出现这个问题是因为你的脚本文件在windows下编辑过且保存方式不当。windows下，每一行的结尾是\n\r，而在[linux](https://so.csdn.net/so/search?from=pc_blog_highlight&q=linux)下文件的结尾是\n，那么你在windows下编辑过的文件在linux下打开看的时候每一行的结尾就会多出来一个字符\r

解决方式

```sh
 sed -i 's/\r$//' csi/update_secret.sh 
```



Q: scp

以下是[scp命令](https://so.csdn.net/so/search?q=scp命令&spm=1001.2101.3001.7020)常用的几个选项：

-C - 这会在复制过程中压缩文件或目录。

-P - 如果默认 SSH 端口不是 22，则使用此选项指定 SSH 端口。

-r - 此选项递归复制目录及其内容。

-p - 保留文件的访问和修改时间。



```shell
scp [option] /path user@server-ip:/root
```

