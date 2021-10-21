# git clone 大项目（apollo）



 git clone https://gitee.com/guopu/apollo.git 

 报错如下

Cloning into 'apollo'...
remote: Enumerating objects: 272519, done.
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
error: 7046 bytes of body are still expected
fetch-pack: unexpected disconnect while reading sideband packet
fatal: early EOF
fatal: fetch-pack: invalid index-pack output



Git异常The remote end hung up unexpectedly解决方案

$ git config --global http.lowSpeedLimit 0

$ git config --global http.lowSpeedTime 999999

$ git config --global http.postBuffer 8048576000 



## 原因：

拉取的项目过大，而使用的http方式拉取，缓存区过小

 

## 解决步骤：

### 一.设置git的全局缓存区大小

1.增大缓存区大小

【在任意一个已存在的项目的根目录下，打开Git Bash。执行如下命令】

```
git config --global http.postBuffer 1048576000
```

单位B
1048576000 = 1000B*1024*1024 = 1000M

 

2.查看设置得缓存大小

```
git config --list | grep postbuffer
```

![img](https://img2018.cnblogs.com/blog/978388/201912/978388-20191223151452635-829013100.png)

 

如果还没有解决

https://gitee.com/guopu/apollo

将https地址换成ssh地址来clone，前前提是你配置好了ssh秘钥，配置方法每个git平台都会有教程的。 一般来说https的方式容易遇到此问题，而ssh的方式不会。所以可以这样解决



