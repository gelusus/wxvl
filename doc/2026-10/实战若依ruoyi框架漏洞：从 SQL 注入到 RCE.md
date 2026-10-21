#  实战若依ruoyi框架漏洞：从 SQL 注入到 RCE  
原创 炫彩之星
                        炫彩之星  菜鸟学安全   2026-10-21 12:48  
  
### 一、前言  
  
　　最近在挖某src的时候，碰到了一套若依，本以为啥都没有，结果随手一测若依存在的历史漏洞基本都有。是通过很奇怪的信息收集方式找到这个站的，分享出来让大家看看。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_jpg/oFamj4ye1grkfM9XVHaOdUWCtHLgIicvzKWSmFAKLF7ia52MnO3r5vU6ThtWyjTBgMxzTnowOFuxSPXZNSr1QCrtico9V1y7bQDh1SlTUuia1S0/640?wx_fmt=jpeg&from=appmsg "")  
  
　　老司机一看就明白了我在找什么，虽然说机会不大，但偏偏就让我遇到了。  
### 二、ruoyi（若依）框架  
  
　　若依（Ruoyi）框架是一款基于 Spring Boot 2.5.5、Spring Cloud 2020.0、OAuth2 与 JWT 鉴权等核心技术，同时也支持Spring Security、Apache Shiro 等多种安全框架，以及 Mybatis、JPA 等流行持久化框架，提供了许多常用的功能模块，包括系统管理、监控管理、任务调度、 代码生成 、文件上传、高德地图等功能的快速开发平台。  
  
　　若依框架采用前后端分离的模式，基于Vue.js实现了前端UI框架，采用Feign作为服务调用，通过 Nacos 实现统一配置管理，是一款高效率、低封装、面向前端的开发框架。  
  
**fofa语句：app=“若依-管理系统”**  
### 三、ruoyi（若依）漏洞  
  
　　前端存储账号密码  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oFamj4ye1gpVeZQOvO2FSbgWhrR2Lg9OOPIV6GHWruBsYI2F5zV2NwZ9daicZyGs2CFcUWZEGWoEuw379QAJob2viaHkHAGK1DOKo4RgYv6iaU/640?wx_fmt=jpeg&from=appmsg "")  
  
　　这个站进来是这样的，没错就是账号密码存储在前端，本来想试试弱口令的，这账号密码都摆着了，（狗头）  
  
　　不过大家遇见若依一般是不会有这种情况的，下面是若依系统常见的弱口令：  
  
admin/admin123  
  
ry/admin123  
  
ruoyi/admin123  
  
　　识别若依框架也很简单，大概就长这样，如果验证码是简单计算，存在记住密码也可能是若依  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oFamj4ye1gqpaD7L2MKDQUPnL40BypXUCGmmzJ7NGZphNibLyICMkufBXia0H1icXGWHUqGJNnkCYGHZs0HnA9MRI3bg4pfJCmMa67nHBHcYeM/640?wx_fmt=jpeg&from=appmsg "")  
#### 1. SQL注入1  
  
　　/system/role/list端点存在注入  
  
POC：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_jpg/oFamj4ye1grC95DeQEr4WjqyPHcTzwRAZRex8EqbpFicAR6QypLWhKI57sT9iad1fm76ZJXzCqnaQvkQw2730oruQpUvwy23hAMaCicSbPFpdQ/640?wx_fmt=jpeg&from=appmsg "")  
#### 2. SQL注入2  
  
　　/system/dept/edit此接口存在SQL注入  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oFamj4ye1gpek2erBhicqzicnGFdibmNHohAsnPKqtbgu8VmoeHUZ8C1Tic35Xf25oRDaMg4rZMibZ2BbW6rqZPd3iciaGFA6BwPBwwa6iacTia2KL8g/640?wx_fmt=jpeg&from=appmsg "")  
#### 3. SQL注入3  
  
　　/system/role/export端点存在注入  
  
poc：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_jpg/oFamj4ye1gos4SQIZcJID92b5tzv5l1eB5MibUvr1cy2OnppiaVSfACHg5fmorF3siajFes0v9dg8TU4AP0gl9HjH4NLL7JBeDJtd1ot5gvOiaQ/640?wx_fmt=jpeg&from=appmsg "")  
#### 4. SQL注入4  
  
　　/tool/gen/createTable端点  
  
　　如果页面如下则可能存在漏洞  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oFamj4ye1gowjrrfgzFS8zFR9Vcicw23ufJSNRjC1XLFu5wmSG8nYNtYDmFGGOOj2htZIMlcmOFoE79U2S3cqTpWjnLR7RypUDCeF0R7v9vc/640?wx_fmt=jpeg&from=appmsg "")  
  
　　这里没有漏洞，我把poc放上来大家可以试试  
```
```  
#### 5. 后台任意文件读取  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oFamj4ye1gpibJ6PQjx2BwHaxM4ibbN7o4ZWFOzm4OJqS36X250SsFo3CTegv7iaBhZ4m14RSslgIZLIp0DGl4QomZfIuGR2hMeUmW40KmaDia4/640?wx_fmt=jpeg&from=appmsg "")  
#### 6. 任意文件读取2  
  
　　漏洞影响：RuoYi<4.7.6  
  
　　首先创建一个定时任务：  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gpBxeyK7yusicoFaGT0kxzxibaHyC2iapJBSGNzh4lsFibq3oR92BpMqXFSuWUjKcIwGukAyyXAiaDNpwKmUBmicFdj6PmDQKWYTTWIs/640?wx_fmt=png&from=appmsg "")  
  
　　执行一边定时任务  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gqvVCiaFbYpSVCaS3Rl1XK56yAqib31SJY9hCVevX7pMmeEhAvy8BBiaozVmSDvHLx3EUxbSuPK3zT48T25w3R4yEI3ia9pz0AECK0/640?wx_fmt=png&from=appmsg "")  
  
　　下载任意文件，可获得前面读取的文件内容  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grud3UslxU69wC22D67kGBHn31WmxfLibsgXRJDPkHxo4Wa2kqq141FHZS9Jn4E94hBcwczf51FthCRZRUOLGPSQkYUShWtp1yo/640?wx_fmt=png&from=appmsg "")  
#### 7. shiro反序列化  
  
　　若依系统是使用了shiro的，所以可以直接当shiro来打。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oFamj4ye1gqcHh0VXgMn35McXaf37AsRUs8YyNNJns6jshWt2exSEibjk5HROGoA0BARvmw98FUxk8T6DaNnibmHdPibTsJyVJ3icar6yXbKPHg/640?wx_fmt=jpeg&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oFamj4ye1gqo9j3ic4giaJtLJtb9z7ak3uFoJtRbW93uTak3ezicDkpEIsdOOU06D4OFkjqlFdAvYXSicBgHAsUzdmQBgjwElTsGiarI69lIcXqo/640?wx_fmt=jpeg&from=appmsg "")  
  
　　直接一把梭哈  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_jpg/oFamj4ye1gqkH4HfKkUVRHBH9mNUpv0GqDEqrr1ClToeWhGQDdG3qoE8PvCjCwg80Zqu0VQiaHrSqfthADqYBOL47up8tOwoGAuRvrMq0OzE/640?wx_fmt=jpeg&from=appmsg "")  
#### 8. 后台定时任务RCE  
  
　　由于若依后台计划任务处，对于传入的“调用目标字符串”没有任何校验，导致攻击者可以调用任意类、方法及参数触发反射执行命令。影响版本：RuoYi<4.6.2  
  
　　1.下载exp  
  
https://github.com/artsploit/yaml-payload  
  
　　2.修改 AwesomeScriptEngineFactory.java文件，exec里改成要执行的命令  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_jpg/oFamj4ye1gqwUZ9EJJOh7AmHpOAzeebzX9Aiaw3MvryJ448wibKvOayFE08mfLWWyG9S1B4MZlrrbxbBic9NNpkqMj7nBeMFgHCtfFdrBoyNd8/640?wx_fmt=jpeg&from=appmsg "")  
  
　　3.编译  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oFamj4ye1gqgnCkvMOZnIpsSAJEaG1aLLURasrKNYuWssnUj97QG5U14ASoxibaYHquoEfu23a3hQ2vkrKpMqYIib4p52MmBXMbXGWeeq2aE4/640?wx_fmt=jpeg&from=appmsg "")  
  
　　就会生成一个.jar文件，放在vps上，开启一个http服务  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_jpg/oFamj4ye1go6DSZBWia1rynX91s6v3NnuM6HS4NK7ibMNd9c2CNib8TIhcDwdKgn58Z6J6ovW4cKRgfnTaKcFkfR7kTtOib3ibzDbsgic7wefyQxM/640?wx_fmt=jpeg&from=appmsg "")  
  
　　然后回到若依后台，添加一个计划任务，这里直接用工具了  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oFamj4ye1gqnpiaKTPkRQbOz5ficzyxWgLFJBDaSUEUm93IfNO044S6r7JkTxnuIIDpYvCBkHoPIAcjWGvjdqHBUO5TyxIONTLicPHeHWTNgas/640?wx_fmt=jpeg&from=appmsg "")  
#### 9. 后台计划任务2  
  
　　漏洞影响：4.6.2 < Ruoyi < 4.7.2  
  
　　这个版本采用了黑名单限制调用字符串  
```
```  
  
bypass：  
```
```  
#### 10.thymeleaf 模板注入  
  
　　漏洞影响：RuoYi <= v4.7.1  
  
payload1:  
```
```  
  
　　payload部分进行编码  
```
```  
  
payload2:  
```
```  
  
payload3:  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1goicRec62n4jB8gykpAL5vmfWGibfkcgWlPPnT8vqDR5tQbWAkfh1Y9me2Nvn7d2M9bicHzfD18kOnal7TFZqGCIOj4Mocpc0PhYI/640?wx_fmt=png&from=appmsg "")  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpXYGrqbwRMt9VUP5WN1YyooaFFEYaJibUJ3fKqtBbwPUzz1Yzrb555O2laTkYWBiaDztn8ejqbDFIU6dsuzMSvH4BGXRHOyicLFI/640?wx_fmt=png&from=appmsg "")  
#### 11. jdbc template sql注入  
  
　　把要执行的sql语句放入其中：  
```
```  
  
　　进行十六进制编码  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gq4NZBNmOTFgrAAK6p1IW5qUVNJcxiaGCdjobjfuw2d9wqkibSMp7g1QEZTecg3AFOoJho1CFibbvyIRYvjCSv0kIh8EUmxTpg7Kg/640?wx_fmt=png&from=appmsg "")  
  
　　重复执行定时任务，jobid保持一致  
```
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqFh1sjIbTj3A40RWl35aRFr9O8zOjEwTU53ibO3zh0FwLG6eS0koTWtXgkyESTWBSJNFEksGZwNaia7SrnjGWJXicpJ0b2RQBNQM/640?wx_fmt=png&from=appmsg "")  
```
```  
  
　　重复执行定时任务  
```
```  
  
　　重复执行定时任务  
  
　　访问  
  
http://192.168.0.129:8088/monitor/job/detail/8  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gpB4eUdZJtep6fb1AY9Wwejpo7NMJSw5WKAE5HojjWwtvojeeA2Dh9ibANA5YwNicb5srnXOmRRyTrqnrCu89PB336BzOpEYm8aQ/640?wx_fmt=png&from=appmsg "")  
  
　　获取shell  
  
　　两种方法  
  
　　一、用wget下载木马执行命令，或者move到网站目录  
  
　　二、直接反弹shell  
### 四、ruoyi漏洞总结  
  
**若依识别**  
  
　　黑若依 :icon hash="-1231872293  
  
　　绿若依 :icon hash="706913071”  
  
　　body=" 请通过前端地址访 "  
  
　　body=" 认证失败，无法访问系统资源 "  
  
　　如果页面访问显示不正常，可添加默认访问路径尝试是否显示正常  
  
/login?redirect=%2Findex  
  
/xxx/login?redirect=%2Findex  
  
　　图标和登录页面及验证码处的样式识别出若依  
  
**未授权访问**  
  
**未授权接口查找：**  
 js 下面有个 app.xxxxxx(一些个字母数字).js，直接里面搜  
  
　　baseurl 就有了，有的是全的 url，大多数是只给个 baseurl：prod-api，这类  
  
　　的拼接就好。  
  
　　之后在接口后拼接  
  
   
![](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqPiaHxqfquSexwfeU8wuCMECibaMr7BIUsunrEEgfyIpr2NnBHTac3o5LDAnaevNq3hCibzGL0KbTiajGNIwABqEJK6UkAyePj6SI/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqp7ZjyPiaLXM6PB95GSxSUY2JwMuD39jibumZ51mbrc4C39f9K3xqzE0LLhfia5vb7ickkkJjRccSXucRmjmcyWuum6Ad8fnVAdR4/640?wx_fmt=png&from=appmsg "")  
  
   
  
　　没查到的情况下尝尝试拼接经典接口路径：  
  
/prod-api/druid/login.html  
  
/dev-api/druid/login.html  
  
/api/druid/login.html  
  
/admin/druid/login.html  
  
/admin-api/druid/login.html  
  
　　尝试弱口令进入后着重观察：  
  
/druid/weburi.html  
  
/druid/websession.html  
  
　　在 URI 监控中存在大量接口路径，可通过未授权获取更多敏感信息在 session 中可获取  
  
　　历史最近的 session 值，可用 session 进行替换实现登录系统。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqgiaOwibIb5pAx1urSe1H6qU07s2OHOwsHkibzrxTrA9WJ28ia4VyqCKzX3VedAPUfVBiaRarA8gXhQtXcZIlPicTSaFPZfhck4htl0/640?wx_fmt=png&from=appmsg "")  
  
   
  
**Swagger**  
**未授权：**  
 在接口后面拼接经典路径：  
  
/swagger-ui/index.html  
  
/v2/api-docs  
  
/v3/api-docs  
  
**Redis**  
**未授权访问漏洞**  
**(**  
**端口**  
**: 6379)**  
  
　　若依会用到 mysql 和 redis 数据库， redis 数据库存在未授权访问  
  
**zookeeper**  
**未授权访问漏洞**  
**(**  
**端口**  
**2181)**  
  
　　里面有可能也会出现 zookeeper 集群部署  
  
**shiro 反序列化漏洞**  
  
　　若依用到了 rememberMe 字段，如果默认 key 值未修改可实现 shiro 反序列化漏洞。  
  
**逻辑缺陷漏洞配合弱口令**  
  
　　若依有个特点，密码可能在前台直接显示，点击并能够成功登录  
  
　　配合弱口令： admin/admin123 ry/admin123  
  
　　Druid 经典弱口令： ruoyi/123456  
  
       admin/123456  
  
       admin/admin  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grmibZFJiamXGUkTZdJicibvYibPCBAJ2U9wnqkHgXeHk2QuyYibHgiciaI2etXMOSoW7hugZ0xENCzAWoC7gyYRYqTYQTtDiab6ibViad2rc/640?wx_fmt=png&from=appmsg "")  
  
