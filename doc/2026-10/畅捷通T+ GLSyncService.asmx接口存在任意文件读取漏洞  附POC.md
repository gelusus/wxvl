#  畅捷通T+ GLSyncService.asmx接口存在任意文件读取漏洞  附POC  
2026-10-19更新
                    2026-10-19更新  南风漏洞复现文库   2026-10-19 15:34  
  
   
  
#   
  
免责声明：请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，所产生的一切不良后果与文章作者无关。该文章仅供学习用途使用。  
## 1. 畅捷通T+ 简介  
  
畅捷通T+专属云适用于需要一体化管理的企业，财务管理、业务管理、零售管理、生产管理、物流管理、移动仓管、营销管理、委外加工等人财货客一体化管理。  
## 2.漏洞描述  
  
畅捷通T+专属云适用于需要一体化管理的企业,财务管理、业务管理、零售管理、生产管理、物流管理、移动仓管、营销管理、委外加工等人财货客一体化管理。畅捷通T+ GLSyncService.asmx接口存在任意文件读取漏洞。  
  
CVE编号:  
  
CNNVD编号:  
  
CNVD编号:  
## 3.影响版本  
  
畅捷通T+  
![畅捷通T+ GLSyncService.asmx接口存在任意文件读取漏洞](https://mmbiz.qpic.cn/sz_mmbiz_png/b9KQYsB8q6ymThibz8vemntYibz5PUq6xzvFFPl4kUlbf1IHDjpZHAYEldc9gUVoY1zweks7hrffBIBULfWKRVGBTbzRXdqu2HeSl7wy3Oxh0/640?wx_fmt=png&from=appmsg "")  
  
畅捷通T+ GLSyncService.asmx接口存在任意文件读取漏洞  
## 4.fofa查询语句  
  
app="畅捷通-TPlus"  
## 5.漏洞复现  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6x57RPxVspAibr7P8iaep09gxqasB5MVmWibXpoyE24pI9tVwiciazLObriaXMsia8DbcTaSZ3H42wFvPTgvQGD5iazoFSAsS9UzPfDESw/640?wx_fmt=jpeg&from=appmsg "")  
  
返回的数据包是base64编码  
![](https://mmbiz.qpic.cn/mmbiz_png/b9KQYsB8q6xCSqTk4QGiaEv1rZdI9kRrvAIpphuj8nb2YnmkicoUzvhkCe8eQJ0PDNWS7TynY1zRn4hiaSOUXqIXYbaxiaNoQHOicRPq4FlYKKJo/640?wx_fmt=png&from=appmsg "")  
## 6.POC&EXP  
  
本期漏洞及往期漏洞的批量扫描POC及POC工具箱已经上传知识星球：南风网络安全  
1: 更新poc批量扫描软件，承诺，一周更新8-14个插件吧，我会优先写使用量比较大程序漏洞。  
2: 免登录，免费fofa查询。   
3: 更新其他实用网络安全工具项目。  
4: 免费指纹识别，持续更新指纹库。  
5: Nuclei脚本。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6xovc9uZs4HiaUMeBG9t5R3QPzr5vbrs4zL0icKRR6cSvjGvJB4MAUQ5wawFHwnEaCl38uajH862iabLhSM9iajJLSG8sUYd0ic5MSQ/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6xHrHicF7Sez7hFM4rSADM13QrCK5ZFsicUdKk9kgSycsn5Uf8Doe6en2okZSv0q2eKiatRM0eA9fABiaK0nxyLx3TibZIUZfdeRm70/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6xNV12nDJKhxcvdiboAibQQDXulUoCqLWYaQyxAuCnjtDjNicsrAG5icMgvEF4T8QyoFVb3Y1iaticHrDExbkibYfYRGhiaCILCcApW8Vg/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6xvictV0NMFI7sAVs7D0wicPMzylBrZZHCa0KJkkfiaclhBXiasdiaQWwv6G65AIfia4sqSXGDbGjOCtefbjDZa5PWhqZeAWv0T13as4/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6zSBxeLW0Cdic7k0S8hNZHux9wlfdEpzADicOKvhPOhYJaUDODpXKIY4tqPadeUMfCkCtXlUmdrEr2vOq00IEPgZsbrOeBEMQXno/640?wx_fmt=jpeg&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6yymP6y80dia2kCtHXIaybxtk0W0KLRqFIetIcu2HjgoqwKxUThomE62r61juvrbDKw7hF4Ud5tuJUMzoCicZK629icCXbCvkpthk/640?wx_fmt=jpeg&from=appmsg "")  
## 7.整改意见  
  
目前厂商尚未提供相关漏洞补丁链接，请关注厂商主页及时更新： https://www.chanjet.com/  
## 8.往期回顾  
  
  
   
  
  
[Atlassian Jira存在任意文件读取漏洞CVE-2026-21589  附POC](https://mp.weixin.qq.com/s?__biz=MzIxMjEzMDkyMA==&mid=2247490884&idx=1&sn=5a1b72d46976e6746a5fd3f3361faffa&scene=21#wechat_redirect)  
  
  
