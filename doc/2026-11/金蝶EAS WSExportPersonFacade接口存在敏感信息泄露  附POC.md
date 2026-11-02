#  金蝶EAS WSExportPersonFacade接口存在敏感信息泄露  附POC  
2026-11-2更新
                    2026-11-2更新  南风漏洞复现文库   2026-11-02 15:54  
  
   
  
#   
  
免责声明：请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，所产生的一切不良后果与文章作者无关。该文章仅供学习用途使用。  
## 1. 金蝶EAS简介  
  
微信公众号搜索：南风漏洞复现文库  
  
该文章 南风漏洞复现文库 公众号首发  
  
金蝶EAS Cloud采购管理系统是专业的采购平台管理系统  
## 2.漏洞描述  
  
金蝶EAS 为集团型企业提供功能全面、性能稳定、扩展性强的数字化平台，帮助企业链接外部产业链上下游，实现信息共享、风险共担，优化生态圈资源配置，构筑产业生态的护城河，同时打通企业内部价值链的数据链条，实现数据不落地，管理无断点，支撑“横向到边”财务业务的一体化协同和“纵向到底”集团战略的一体化管控。金蝶EAS WSExportPersonFacade接口存在敏感信息泄露。  
  
CVE编号:  
  
CNNVD编号:  
  
CNVD编号:  
## 3.影响版本  
  
金蝶EAS  
  
![金蝶EAS WSExportPersonFacade接口存在敏感信息泄露](https://mmbiz.qpic.cn/mmbiz_png/b9KQYsB8q6z7F2e3ntuy39HJmAPJan0RyicWQw9pS0VUWwPgXtbiaMRNhIBOEMsKBCeErMlaFtJq43KIKAnLXlu5gePnycSOsj7RMapSUD61Q/640?wx_fmt=png&from=appmsg "null")  
  
金蝶EAS WSExportPersonFacade接口存在敏感信息泄露  
## 4.fofa查询语句  
  
body="easSessionId" || body="loginCenter.jsp" || title="EAS系统登录"  
## 5.漏洞复现  
  
漏洞链接：http://xx.xx.xx.xx/ormrpc/services/WSExportPersonFacade  
  
漏洞数据包：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6yibBxgVr3TicOiaxcBTW3RpgMVeIicPQKngYpm2rXKQKZqD7icuzVuBXmqJgtMzpnYFga1HG3A4qZG46y1p4oRmV07NxLH10ibBTPh4/640?wx_fmt=jpeg&from=appmsg "null")  
  
## 6.POC&EXP  
  
本期漏洞及往期漏洞的批量扫描POC及POC工具箱已经上传知识星球：南风网络安全  
  
  
1: 更新poc批量扫描软件，承诺，一周更新8-14个插件吧，我会优先写使用量比较大程序漏洞。  
  
  
2: 免登录，免费fofa查询。   
  
  
3: 更新其他实用网络安全工具项目。  
  
  
4: 免费指纹识别，持续更新指纹库。  
  
  
5: Nuclei脚本。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6yNlMhRNg92ca0icjlOygEnLrCBLtCsXEYZicIGIv3uzG5bKicIwoOTFCA4H5RLib0OLnzVWbJqBjpyjOFcJ1ibibu756gv05reXTZd8/640?wx_fmt=jpeg&from=appmsg "null")  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6wqaY3kDgn3ZI5VTZDg0SdqmQfP247UKoA2ejRS85rz5gAGjzbHRg1S7B9hmhWgxaDBpiaULvYPsHLyZpdibTIdN9yHK1KMTCYaY/640?wx_fmt=jpeg&from=appmsg "null")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/b9KQYsB8q6yic9hPqodhoZAwTCTk3TNAfgBus5UORFTmnOVUp057eXl6AdpUzXUAoZy6F69Jd9dpL0icVUAcSGGspLqn3J53t3ONeG63sdDAI/640?wx_fmt=jpeg&from=appmsg "null")  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6x5X8hY7ugI97QmkmcW073sDdcEVJ8ekPr3NHEZaIKkOFo90fbBzm6Rn0flndn9qR2Fcibq4vbrKDSiaaZiaIx6XlIuPoyXAvkic5I/640?wx_fmt=jpeg&from=appmsg "null")  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6zQQTq5a9q31ryVAic32CCEtcxJbYoF4AyZTgVBNLTyaLpJIXa8BA9j9JFhn48Zcg6PuoSpZyEmY16BVMYvmXdG6SibOw23fJQ4A/640?wx_fmt=jpeg&from=appmsg "null")  
  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9KQYsB8q6zHy1TdXIQcjXGIGtLCKtsdgCiarm4yK7vkict2CnibJZ7gkYRBoibiaOcAtQG3OiaBN3TLz6BKQBtnpWfZqbd2JyAEpWlR1b53d9fzI/640?wx_fmt=jpeg&from=appmsg "null")  
  
## 7.整改意见  
  
请联系官方打补丁：https://www.kingdee.com/  
## 8.往期回顾  
  
  
   
  
  
[InvoicePlane存在任意文件读取漏洞  附POC](https://mp.weixin.qq.com/s?__biz=MzIxMjEzMDkyMA==&mid=2247490976&idx=1&sn=2fc5645f6d919e52779f330ecff4dddc&scene=21#wechat_redirect)  
  
  
