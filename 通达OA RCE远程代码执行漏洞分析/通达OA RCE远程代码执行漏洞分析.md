# 通达OA RCE远程代码执行漏洞分析

# 前记
&nbsp;&nbsp;&nbsp;&nbsp;在上个月，通达OA爆出了任意文件上传及任意文件包含漏洞，影响版本非常广，这两个漏洞结合可以获取shell权限，为什么在漏洞爆出一个月后我才写这篇文章呢，因为我也是最近才看到的。所以我也赶忙的写完EXP和POC就发了这篇文章，下面我会从代码层进行审计，分析漏洞。希望对大家有所帮助！

**0x02 影响版本**
tongdaOA V11
tangdaOA 2017
tangdaOA 2016
tangdaOA 2015
tangdaOA 2013 增强版
tangdaOA 2013
# 任意文件上传分析
&nbsp;&nbsp;&nbsp;&nbsp;该文件上传漏洞的产生位置就是在upload.php文件中，因为其未对用户身份进行严格认证导致任意文件上传漏洞，下面我们来看一下吧。

&nbsp;&nbsp;&nbsp;&nbsp;源码采用了zend加密，解密后才能正常阅读代码，从下图中我们可以看到代码对变量P进行了判断，如果变量P被设置并且不为空则获取session否则进行身份验证(从文件名不难判断)。这里值得吐槽的一点是既然使用了isset函数进行判断，为什么又要在后面检查P是否为空，这样不是多此一举吗？
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427133635311.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
继续向下看，要求$DEST_UID不为空，后面调用了方法td_verify_ids是说这个参数需要是0-9开头的值。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042713524871.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)

下图框内代码的意思是如果DEST_UID的值为0那么UPLOAD_MODE的值必须是2否则返回接收方ID无效。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427135627891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
继续向下看，判断是否有文件进行上传，然后对UPLOAD_MODE的值进行判断，如果不为1则进行调用upload函数，否则会进入if语句中进行判断上传的名url解码后与解码前是否一致。upload函数位于inc/utility_file.php中，所以我们继续跟进该文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427135806766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
我们进入inc/utility_file.php中发现其中调用了is_uploadable()函数对文件吗进行验证。如果文件后缀名为php则返回false，返回false的结果自然就是禁止上传，这里要注意。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427141300195.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
从上面的审计中我们得知了一些必须提交的参数，好在这些参数都是我们可控，所以我需要构建一个数据包。如下图提交相应参数，P、UPLOAD_MODE、DEST_UID这三个参数是必须提交的，其中上面有说到的一些判断条件也需要注意。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427142117788.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
现在通过返回值我们得到上传后文件名及路径信息。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427143926719.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)

我们发现已经成功上传，但是上传后的路径并不在web目录下，并没有办法直接访问，所以我们就需要结合另一个文件包含漏洞了。

# 任意文件包含漏洞
&nbsp;&nbsp;&nbsp;&nbsp;在上面我们讲到了任意文件上传，但是并没有办法去作为php代码利用，所以我们需要利用另一个任意文件包含漏洞，该漏洞产生于gateway.php文件中，下面我们来看一下吧！

&nbsp;&nbsp;&nbsp;&nbsp;该文件代码较少，箭头处的意思是接收的json的key值如果是url则会给url变量赋值他的value值，方框中判断接收的url值中是否含有general/、ispirit/、module/任意一个，如果有则直接进行包含url。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427142841366.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
所以我们可以传递一个json值，他的key为url，value值包含general/通过../跳出访问我们之前上传的那个文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427143850920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
构建语句http://192.168.52.133:81/ispirit/interface/gateway.php?json={"url":"/general/../../attach/im/2004/1426921525.jpg"}&cmd=ipconfig
已经成功执行命令，这里需要注意的是通达OA会对一些木马文件进行过滤，比如上传常规的一句话木马是无法执行命令，亲测通过上面调用cmd的方式可以执行且权限为system。
这里有一个笔误，之前上传的木马接收的是POST的值，为了方便展示我改成接收GET了。

# 奇淫巧技
&nbsp;&nbsp;&nbsp;&nbsp;既然上面已经有了文件包含漏洞那么自然我们也可以通过一些其他方法进行利用。比如日志包含来进行代码执行，下面我们来看一下吧。

&nbsp;&nbsp;&nbsp;&nbsp;在GET请求处写入一段php代码，注意一定要抓包后写，如果直接在浏览器进行构建语句，则会被url编码，写进日志自然无法执行代码语句。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427150135665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
下面我们进行包含日志文件吧。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427150201463.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
可以看到代码已经成功执行，同理oa.error.log也可以进行文件包含。但是需要注意的是构建php语句时不要输入双引号，会被日志进行转码，从而无法通过日志getshell。要使用单引号

再就是文件上传中的一点奇淫巧技。
在构建上传数据包时，将文件名后加一个.利用windows的特性会自动忽略.达到绕过上传文件限制，但是因为上传后的路径不在web目录下，所以仍然需要与文件包含结合利用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427150704229.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
这里看到上传后确为php文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427150829878.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
这里顺带讲一种文件上传的利用方式，利用phar://协议与文件包含结合绕过。首先创建一个文件名为test.php，文件内容如下:

```php
<?php @system($_POST['cmd']);?>
```
然后对其进行压缩，再将压缩包后缀名改为jpg
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200430120910729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
内容如上图，通过文件头相必小伙伴也能看出本质上依然是一个zip压缩包。
这里值得一提的是这个漏洞需要与文件包含一起使用。这里为了方便演示直接创建一个包含文件，内容如下:

```php
<?php include('phar://./test.jpg/test.php')?>
```
在实际环境中，想要利用通过include中方式包含可以getshell。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200430122152288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
如图，我们发现已经成功getshell，通过上面的一些方法其实相信你已经若有所思了。

# 官方修复补丁
&nbsp;&nbsp;&nbsp;&nbsp;在官方的修复补丁中，对文件上传进行了删除else判断，如下图，因为我不想打补丁所以就从网上找了一张对比图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427151027540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
可以看到原先是进行验证变量P，如果符合要求就不会进入auth文件中，修复后无论如何都会进入auth.php文件中进行身份验证。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427151146803.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
而文件包含处则对..进行了过滤，不允许url中包含..从而禁止目录穿越。

**修复方案:
官方已经发布了修复补丁，可以通过安装补丁解决。**

# POC&EXP利用
&nbsp;

**POC使用方法**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427151923488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
POC的使用方法比较简单，只需要指定域名即可自动检测是否存在该漏洞。

**EXP使用方法**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427152148223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
默认指定域名则会返回一个交互式命令执行终端，输出exit退出交互。![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427152437852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
如果指定-file-shell参数则会自动生成并返回一个木马连接路径，访问密码是pass，通过冰蝎进行连接，注意菜刀这些都不能正常连接，要用冰蝎(如下图)。![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427152626876.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)

# 后记
&nbsp;&nbsp;&nbsp;&nbsp;那么通达OA的远程代码执行漏洞就讲到这里了，上文中从代码层分析了漏洞的成因，对于网络安全测试中，不能只会渗透而不会如何加固，只有从代码层分析漏洞成因才能从根本上解决。过一阵我可能会写一个通过搜索引擎批量利用的exp，会发出来大家可以留意一下。希望看完文章的你能从中有所收获。
&nbsp;&nbsp;&nbsp;&nbsp;**愿有心人皆有所成，所想之事皆能如意吧**