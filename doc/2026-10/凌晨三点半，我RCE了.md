#  凌晨三点半，我RCE了  
ptr
                    ptr  UpRoot   2026-10-10 04:24  
  
我给自己定了个目标，想尝试一下集权设备产品的前台rce。  
  
这个产品的3.0.12，我能够通过伪造会话的方式去rce，我当时高兴了一段时间，后来我去打了补丁，才发现3.0.12的202504版本就已经修了伪造会话的接口。这个版本过后，前台几乎没什么能够操作的接口了，或者说应该是我目前能力还是不够，没有找到能够前台rce的点。  
  
经过几天的审计后，今天凌晨三点多，我rce了，但可惜他需要个会话，或许以后打项目时，浏览器抓凭据或者运气好拿到普通用户的凭据时，能够想到运维段还有这么个安全设备能够梭哈完成任务吧...  
  
漏洞思路：  
  
组合拳：最低权限账号（任意账号）-> 任意读 -> 数据库文件中的jwt secret -> jwt伪造 -> 权限提升 -> 高权限接口设置属性 -> 建隧道 -> 写文件 -> rce -> 后利用。  
  
环境（最新版也可以打，本地已经升级测试过了）：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/KA5KNdck7pdibErL3rrHWbq3Hq7AicxL42p4N5347dCY4ibOw6nqbTaMZBkE7NG1rPBbVibft3xyTrvJ4khZQjojAl9eiaISWxdJ2pnFh09P2ATA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/KA5KNdck7pcc4iaicG94vXxiaewvUpoxvsTlSyuMgRHe2NPeUnlxZ1flzTbKXwjquKO3v2HO0E9TnJTQqsq5kJuETGoRian7WFLompTSbVVfq2I/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/KA5KNdck7pcZdgnMkHuo2ibyBy6Qh6IIpMIqZQ5nB4rvpQsoVhyVuYvVMe2fn8Xia9SU4U81ATUUePfSqK62yicRCRxGeoUSoIjkEqUFbGIMOQ/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/KA5KNdck7pfEydwBZR8Q0MQxwP6A7eYsHz9icEmlwWN2mXPxBwLgm8ZhcMQnQG2DSOuftptCHlWeLNLSfsgLlvQWj8lnbib8EVnj26XUTy2ib8/640?wx_fmt=png&from=appmsg "")  
  
最近有的师傅经常在后台私信问我职业发展的问题，其实我想说我确实给不了什么建议，我也正在走一段上坡路，也在尝试去把一些事情做好，所以我不想通过我的主观判断去影响大家的以后的路怎么走，况且正年轻，多尝试多试错，然后总结反思，想下一步该怎么去走，我觉得才是我们这个年纪该有的。  
  
焦虑也好、迷茫也罢，不要放弃，很多过程上的失败，都或许是在为最终那个更大的结果蓄力...  
  
- END -  
  
  
