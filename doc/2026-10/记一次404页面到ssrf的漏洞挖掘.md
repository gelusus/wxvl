#  记一次404页面到ssrf的漏洞挖掘  
原创 ss
                    ss  shadowsec   2026-10-18 05:41  
  
    说到404，最常见的就是springboot的whitelabel error pa  
ge了。  
这个页面并不代表服务不可用，恰恰相反，它说明 SpringBoot 应用已经部署启动成功，只是当前请求没有匹配到对应的路由。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/GjcPTs2YKv7ibbOZdX1FVP61gBUS1MV9ypNRBKlz50yATUvicIZycsGDfiaN2QqeNXMicPDfIEA2OJT4l76Pr17n3mnA0ia47fffwxCQ7VE3iblW4/640?wx_fmt=png&from=appmsg "")  
  
    拼接上常见spring接口，测试地图就到手了。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/GjcPTs2YKv5dnTZibWia1UKDsM0fa0rvQceemCFrCdIMzoARQPUY2vg6LEULcFdx7k0HO0Dlu4sty19xCtkcIyOib1qEVxjsqzxyPahfjhURMc/640?wx_fmt=png&from=appmsg "")  
  
     
 到这里就能发现：404不只是一个简单的HTTP状态码，不同场景下返回的404，会透出  
完全  
不一样的信息  
。比如有的  
直接回显Whitelabel，说明应用活着只是缺  
路由。  
有的响应里回{"code":404,"upstream":"..."}这种带upstream 字段的JSON  
，说  
明  
请求  
命中了后端/xxx，  
只是  
Host头/域名校验未通过  
，导致被网关拦截。有的直接回显的  
Tomcat  
,  
IIS  
的版本号，很明显是容器行为，可以顺着版本尝试nday。当然也有无任何信息的，大概率做的是  
反代  
（  
这里借用下截图）  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/GjcPTs2YKv5FicibqWmyTKhHcxU5xZgnu5H2fkiarmdARO9OY0mPp6L9MBmABoffae0ZPFp0LY31NAU6LMQDGJdyUQyWY7rSZf77pWU36MbW9I/640?wx_fmt=png&from=appmsg "")  
  
    
  针对 Nginx 这类无特征的  
404，  
可  
以用Google、Bing  
的  
site: 语法去找被收录的url。再逐个访问，看哪  
些还能打到真实业务  
  
![](https://mmbiz.qpic.cn/mmbiz_png/GjcPTs2YKv7CSibqW8sFAJgPJf0JYqsQNXO0E5S2HmhcRywFgmm4HXySbQsZPOPH7nUQqqD77sck2yNHnbdMAic9cic6ydyI4yPp710rYZq6bM/640?wx_fmt=png&from=appmsg "")  
  
    也可  
以通过wayback machine回溯历史快照，抓取历史js文件，访问路径，robots.tx  
t记录。  
```
https://web.archive.org/cdx/search/cdx?url=example.com/*&fl=original,timestamp,statuscode&collapse=urlkey
```  
  
      
如果这些都没有收获，再用精简字典补一轮。  
  
    最近和表哥渗透遇到个系统，用工具扫完之后筛资产，前期大部分子域名都没什么收获，最后转向一个只回 404的页面，想再挖一挖。  
访问域  
名返回：  
```
{"code":404,"message":"Not Found"}
```  
  
    正常的404页面基本都是html，这里回的是json，第一感觉是它接了统一的异常处理。抓包  
观察  
响应  
，发现里面携带了多个自定义响应头，其中一个自定义头直接泄露了网关产品名称。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/GjcPTs2YKv6RBuFiaJ8AWPcFHq4uIRbnZGjF5Kpxsr5rXKAm337k6qrLMfEQnSzo8BSehiaPryUI5aF8GKhS0A76WFPxxGof79YDvaLWvUUqs/640?wx_fmt=png&from=appmsg "")  
  
   平时遇到比较通用的可能就先跳过了，但是这个很小众。通过网关名进行互联网检索，在csdn一篇文章中发现在过去该网关作为试验产品开源过，且邀请共同维护并附上了开源github链接。说明是自研且有源码  
  
    访问github链接代码仓库已经清理  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/GjcPTs2YKv5iaOqmnls7NxX8ibWwLMwnMHDDpmlxAWa6zCUFQl1v9cF2EXuXwOVdlLQhvQwF2dRHGnDj838hvBX7Y1kIvwuGQkhskic9Mbx0PE/640?wx_fmt=png&from=appmsg "")  
  
    但原仓库清理，过去一起维护的其他人呢？面对这种产品我一直相信有人会fork下来。通过多角度搜索github仓库，终于在一个角落发现了一个fork版本。  
把源码拉下来和线上对了一下，发现只有少量接口对得上。也就是说后续厂家做了很多轮更新。  
那这条线就断了吗？  
  
     
 通过收集少量代码中的少量接口进行fuzz，发现几个  
未授权的存在，但是不返回内容。此时再思考网关的存在大概率是需要监控的，监控健康状态、指标等等。那么利用ai生成几个监控常见路径字段，如  
metric，  
health，status  
。批量fuzz之后最终在某个未授权路径下的/metric成功获取了监控的路由整表  
。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/GjcPTs2YKv5A6iblaoQa4PHSqMNfnHjCibVz8mczhJYQIBJTTREkKebgxrKdCbOYgNdY78khiaKcsbHIDR1lnaMUiaJEnaC85lojDBCujOlme44/640?wx_fmt=png&from=appmsg "")  
  
    那么这下好办了，拿到全量路由批量探测。但基本都是401状态，即使是响应200的也没返回什么内容。  
  
    又重新仔细看了下url，然后发现了一个/xxx/xxx/xxproxy?url=的接口，遇到这种接口很明显，代理url一般就是ssrf高发接口。探针  
```
?url=http://xx.xx.xx.com/latest/meta-data
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/GjcPTs2YKv55dKrD4LAdTyJH8ZJTkbzdHkbgurSFqwb6e6uMBItria0a7WosCpNVCLoQBPZwJx6iaEWtlu6766I7UzFxEdP2ibFvUzawXARaMU/640?wx_fmt=png&from=appmsg "")  
  
最终获取了元数据。  
  
   
     
这个案例也让我重新审视自己过去的资产筛选规则。  
之前面对工具扫描出的成千上万的资产。  
为了快速把有价值的  
挑出来  
，所以最  
初  
我在第一阶段设定规则就是  
响应200、存活、有内容。命中的进入下一轮，其他的就丢弃，以至于很多404等资产被忽略。  
  
    过去精力有限，一天只能测试几个系统，大部分人都一样，所以成熟的流水线也能让我挖出漏洞。扫全量 JS 与懒加载 chunk，还原 Webpack打包，尽可能捞出接口后批量进入burp测试。而现在AI辅助测试之后，这一流水线速度提升巨大，且更细致。之前的脚本还依靠正  
则可能会稍微遗漏，要分析前端加解密需要调试花费长时间。现在js面已经扫描的非常干净了，且加解密基本能秒解。速度上来了，同样的时间能覆盖的资产也成倍增加。依照旧规则获取的业务系统很大概率已经被别人扫描测试，再加上  
打法又高度趋同  
，那么就只剩下  
拼手速。在这样的情况下，很明显之前资产收集的规则需要重新规划，挖掘更多的隐藏资产。之前的404等资产也应该接入到目标资产里来，不怕杂更怕少，杂不烧心。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/GjcPTs2YKv5E0LgGAaibm9cicN6JxFSgQMJTCtKOoQk6vLOSrQ9iavIAAznmic2nLyfaP5hSXacDSrSHyqIibHsR1kcGZloZouxjfSibJCEjQAyaU/640?wx_fmt=png&from=appmsg "")  
  
