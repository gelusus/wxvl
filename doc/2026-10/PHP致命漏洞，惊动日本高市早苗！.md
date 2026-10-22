#  PHP致命漏洞，惊动日本高市早苗！  
原创 hacking
                    hacking  Hacking黑白红   2026-10-22 09:57  
  
# 昨晚，堪称无数运维和网安人的“不眠夜”。  
  
PHP突然爆出两个CVSS 9.2的致命RCE（远程代码执行）——CVE-2026-109618与CVE-2026-109619，合称PHP2Shell。  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TVljsu2eAicK8qMHg3fIibialPkmPEowGfoSH4C61jdcZvn4E5b0ialTHtuPpqibsLria4LISjbF5y6v8Oq9RpSCmuZdbEEWMPnUZDRdyVhCJma8M/640?wx_fmt=jpeg "")  
  
  
  
这洞太狠了：  
  
不用登录、不用交互，发个恶意请求就能打穿服务器。更致命的是，漏洞在PHP引擎底层，业务代码还没跑，攻击就先触发了。  
  
## 一、惊动日本首相高市早苗，罕见“催更”补丁  
##   
  
这次恐慌到了什么程度？日本政客高市早苗竟在社交平台深夜发推，引发3371万浏览、22万点赞。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/TVljsu2eAicIZ65VQszfSSpnd1w5f5yudOPfZPVI0OSAd22licQxRsDzEiaukwUplz93ICEIYlNfvqwGaMB6DkUJI6A4QYL2eLhwvkEqt771d4/640?wx_fmt=jpeg "")  
  
  
  
  
她紧急呼吁：  
  
“许多网站使用的PHP语言已发布严重漏洞提醒，请确认公开网站是否使用PHP，并为今晚（10月21日）下午11时发布的修复补丁做好准备，以便尽快应用。”  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/TVljsu2eAicJ2UBugd9IBnE7SDDUKwHddnccW9hQmo6mIp1Nt6DT1cia4Q1Q4K19LSm1UrTMqm7NUfPoGFyI9jS1CVAYwFeKrdPYlfUpNBmOI/640?wx_fmt=jpeg "")  
  
  
  
连政客都下场喊话“拔网线”，可见波及面之广。  
  
  
网友评论：  
  
  
“日本估计大量的祖传代码，有没有用上php我都存疑，去年大阪世博会的网站没有前后端分离，每次跳转会把整个网页所有资源重新加载一次”  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TVljsu2eAicL4naYXzEcdSwSLdXxZ2H3NWMrzDBuFDdv7f8DJGyV91nsZtMKShHFBtzLWkD3pOfHFUIicTvvAsMXb1SsVRRqE8ibDRYicgIPLV0/640?wx_fmt=jpeg "")  
  
  
  
## 二、AI把“老水管”炸了  
##   
  
全球约70%网站跑在PHP上，像城市地下的老水管，一爆全街遭殃。这次不是人熬几个月挖出的——Calif拿AI在“修补地球”计划里揪出缺陷。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/TVljsu2eAicKCpyzyO3icRjUZdKoE7jeAicBf0s5l9pQaaNntI9BW2mu4mvjdV8r0Nqz8SyIcrE7q5dwDdllaBUtEL6O2ricZnicwaCNh97cAqBE/640?wx_fmt=jpeg "")  
  
  
  
一个是OOM内存耗尽时的状态错乱，一个是跨请求残留的“释放后使用”。这类洞过去要顶尖高手磨很久，现在AI快速定位，两个月协同修完补丁。  
  
## 三、保命动作别磨蹭  
##   
  
发现只要几天，修复熬了60天，这就是“红皇后赛跑”。AI让“找洞”便宜了，可“修洞的人”没变多。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/TVljsu2eAicITX44HVBOsZsvOo199pYiaV4XLUHpB8mYUx8iaBcib2LbmwOOzdAXzfJp62UHyBRZ41x3zRZfzWnmxibImE0Z8LAaWTgJTgibXSVlg/640?wx_fmt=jpeg "")  
  
  
- 立刻升到修复版本（8.2.35/8.3.36等），重启所有worker；  
  
- 8.1及更早找LTS回补或升新版；  
  
- 临时缓解：设memory_limit=-1并关session.upload_progress，但更新才是唯一解；  
  
- 公网老项目、无人维护后台，能下线先下线。  
  
旧时代的债，终于在新时代连本带利还了。  
  
  
-- END --  
  
*本文作者：hacking  
。  
文章内容或图片来自网络，如果存有侵权，请留言，第一进行处理。  
  
  
