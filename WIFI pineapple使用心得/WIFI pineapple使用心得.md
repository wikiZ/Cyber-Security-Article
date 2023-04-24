# WIFI pineapple使用心得

# 前记
&nbsp;&nbsp;&nbsp;&nbsp;WiFi Pineapple 是由国外无线安全审计公司Hak5开发并售卖的一款无线安全测试神器。集合了一些功能强大的模块，基本可以还原钓鱼攻击的全过程。在学习无线安全时也是一个不错的工具，下面我们来看一下这个工具的使用吧！
# 环境概述
&nbsp;&nbsp;&nbsp;&nbsp;首先拿到这款菠萝派，我们需要先看一下的整体结构。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020033110014259.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
基本上与我们常见的路由器相似,右侧有一个WAN口、USB接口、TF插槽，左侧有一个LED灯、电源接口，重置键。
首次使用我们需要访问172.16.42.1:1471，这是它的管理页面第一次进入需要初始化，这里可以看官方文档我就不多概述了。
他有两种联网方式:有线、无线。
有线:直接将菠萝派的WAN口接入路由器的LAN口。
无线:可以在管理界面中扫描附近WIFI进行连接(如下图)。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331100757700.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
在networking处找到WIFI Client Mode处点击scan进行扫描，再选择wifi输入密码连接。相信大家会疑惑那个USB接口是做什么的？官方的说法是可以进行插入U盘或者连接无线网卡等。最常见的就是拓展一块网卡，可用于 PineAP 或者功能模块，工作在 monitor 模式。

**侦查模式**
&nbsp;&nbsp;&nbsp;&nbsp;我觉得他的一些 选项侦查模式还是比较有讲解意义的，他和我们之前讲得airodump-ng的探测很像，也是对附近wifi进行扫描探测的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331101410538.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
在图中我们可以看到箭头一指向的Recon（侦察），箭头二所指的start正是开启扫描。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331101855932.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
是不是感觉很眼熟，相信看过上一章的你一定有所了解了。但是这里我们还是要讲一下，自左向右看第一列分别是wifi名、MAC地址、加密方式、是否开启WPS、信道、信号强度(数越小信号越好)、最后一次探测到。这里的使用远比airodump要方便不是吗，毕竟是图形化界面，体现的更加简洁，便于阅读。
# 模块使用
&nbsp;&nbsp;&nbsp;&nbsp;在左侧的选项栏中，我们可以注意到Modules，在他的侧面我们可以看到GET modules from WIFIPineapple.com这里是用于获取模块的。![](https://img-blog.csdnimg.cn/20200331102542526.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
我们进入获取模块的选项。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331103054962.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
里面的模块还是挺多，基本上就是我们常见的一些工具，集合在了一起，点击右侧的install即可下载，这里建议大家把模块下载至SD卡，之前介绍的TF接口这里不就派上用场了吗？官方的内置内存只有16MB只是远远不够的，所以我们需要拓展一块SD卡，这里我建议4-8G即可，基本上用不上多大的。

**DWall(绵羊墙)**
&nbsp;&nbsp;&nbsp;&nbsp;开启后这个模块会将连接wifi的客户端访问过的http请求以及cookie和查看过得图片显示出来，效果如下图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331103809718.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
**Deauth（取消验证攻击）**
&nbsp;&nbsp;&nbsp;&nbsp;这里其实就是之前讲过的取消验证攻击，发送一个离线帧，致使线上的用户全部退出登录重新验证的方式，这里我就不演示了，因为这个菠萝派性能一般，之前做的时候崩过。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331104059574.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
这里讲选项配置好，注意保存然后开始即可。

**DNS Spoof（dns欺骗）**
&nbsp;&nbsp;&nbsp;&nbsp;如下图，在hosts栏中进行配置和平时的hosts文件一样编辑即可，图中所示会将www.baidu.com指向172.16.42.1这个ip。Langding Page可以自定义跳转后的页面，需要注意的是这个模块会和其他的一些模块产生冲突。![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331104508226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)

**Evil Portal(邪恶门户攻击)**
&nbsp;&nbsp;&nbsp;&nbsp;该模块开启后，连接菠萝派wifi会自动跳出一个认证界面，认证界面可以自定义，这里我们可以制作一个钓鱼页面，比如QQ空间？首先我们先来看一下这个模块的配置界面。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331105104806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
在这里创建一个门户，然后点击门户名称进入页面配置。默认会有三个页面，这里我们可以自定义页面达到钓鱼的效果。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331105222720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
修改后保存，可以在下面的实时预览查看效果。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331105304499.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
这时我们尝试连接wifi，发现自动跳出这个界面，然后我们输入用户名密码。
![在这里插入图片描述](https://img-blog.csdnimg.cn/202003311054372.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
这里使用Winscp进行连接，Winscp的文件管理非常简洁，类似于windows风格，所以我们比较喜欢用。端口设置为22即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331105645574.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
这里发现密码已经在上面，需要注意的一开始点开可能会出现乱码问题，这里在上方编码处将其改为UTF-8即可。这里钓鱼界面我是别的地方搬来的，大家也可以自己写一个，拿到模板页面后自己写一个IO流写入文件即可。这里不知道为啥我的手机并不能弹出。

**Status(配置显示)**
&nbsp;&nbsp;&nbsp;&nbsp;这个模块我真的非常喜欢了，一些重要配置都在里面，像路由器基本信息和配置，网卡，内部存储空间，连接客户端等。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331110029581.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
RandomRoll(随机卷)
&nbsp;&nbsp;&nbsp;&nbsp;这其实就是个恶搞界面，开启并勾选界面后不管你访问什么网页，只要你连接的是菠萝派这个wifi都会随机跳转至这些界面。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331110420475.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
看一下效果图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331111010449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
# 心得分享
&nbsp;&nbsp;&nbsp;&nbsp;综合来讲呢，这个WIFI pineapple并没有给我太多的惊喜，我感觉他其实就是一个工具合集，但是其实无线钓鱼也就是这么一些东西，作为研究无线攻击来讲我觉得这个工具挺好用，一些工具都清晰简便的集合在了一起。这里我罗列一些需要注意的地方。

 - SD卡，注意这里强烈推荐插入一个SD卡内存不需要太大。
 - 国产菠萝派的信号不好，不要离着太远了。
 - 初始化时可能会存在不成功的现象比如HTTP Error这一类，请多尝试几次。
 - 在管理界面的networking中Hide Open SSID一定不要勾选，不然就没有开放的AP了
 - 信号一共分为两个开放式和有密码的AP，两者除了有无密码暂时没有发现其他区别。

&nbsp;&nbsp;&nbsp;&nbsp;其实还有一些玩法，因为比较多就没有罗列了，只写了一些最基本的使用心得，大家如果有什么不懂的地方可以私信我。

# 后记
&nbsp;&nbsp;&nbsp;&nbsp;不能讲不值，毕竟学到知识才是自己，但是我感觉并没有达到我的预期而已，无线安全其实范围也是比较大的，在学习菠萝派的时候其实也算有一个意外之喜，我了解到一款hack rf的硬件设备，也是属于无线安全，是无线电范畴，我昨天了解了一下，感觉很深，所以我想等着过了这一阵就去研究这个，因为我觉得网安是一个很大的范畴，我想要把一些知识都去了解学习过后，再去确定一个方向深入研究，毕竟不能什么都会，什么都不精对吧？所以最近一阵时间，我就会去复习以前学习的一些知识了，也有一些好久没有再碰的比如java我想要重新拾一下，我怕手生了。接下来的一段时间里我暂时就不会经常更新了，专心复习。

**祝大家前程似锦，心想事成！**