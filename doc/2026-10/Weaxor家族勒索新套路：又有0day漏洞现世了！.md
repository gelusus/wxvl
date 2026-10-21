#  Weaxor家族勒索新套路：又有0day漏洞现世了！  
MSSP运营团队
                    MSSP运营团队  电信云堤   2026-10-21 08:53  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/Dh3fqSPAOWekCSIf3ffuFuiaBPl4BSArBsDhFEMSOTbeIfb7mdz4D0mDExZesv4PPicUdsOTxfRUx8QntAMTmTBA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/Dh3fqSPAOWePMu18J3qdm7D4DgerZrtmzfZ2ThoQZZl4WAHJScyL4c59NPZcjXgnP2YL4HnUicxRibjtBZmqJW8A/640?wx_fmt=other&from=appmsg#imgIndex=1 "")  
  
近期，  
中国电信天翼安全MSSP运营团队监测到  
Weaxor勒索家族  
利用  
xx通ERP系统  
0day漏洞  
进行全网批量扫描攻击，造成主机文件被加密，业务无法正常使用。  
  
**01**  
  
**情报概述**  
  
  
据MSSP平台告警显示，自9月30日起，攻击者利用此漏洞进行全网资产扫描，在国庆期间对大量暴露在公网的xx通ERP系统进行勒索攻击。国庆期间，MSSP运营平台推送的高危勒索攻击安全事件达55条；截至目前，累计安全告警推送已达155条。  
天翼安全MSSP团队特此提醒广大客户：需提高警惕，加固业务系统防护。  
  
十月以来，MSSP应急处置团队已接到超10家客户反馈，其ERP业务遭受勒索攻击，无法正常使用。经应急团队深入分析，确认攻击者利用xx通ERP系统 GetAnonymousTPlusToke 接口的SQL注入漏洞，进行勒索病毒攻击。  
  
**加密后缀**  
**weax****：**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NicWxqLvw6OpqhHkLOWR8cz0Nnatlhq5I2oABKiclAH3Xh225EJ4uiaTiciaNrahXiccSOfMWkibNd4foBmfATPcZT8OqBljZD4x3LJNIKSiadPiaFOE/640?wx_fmt=png&from=appmsg "")  
  
**02**  
  
**MSSP平台日志分析**  
  
  
**1.根据攻击者代理IP，依托MSSP平台日志检索，定位攻击者利用**  
/tplus/api/v2/collaborationapp/GetAnonymousTPlusToken?IsFree=1  
接口，持续对大量使用xx通ERP系统的客户进行勒索攻击。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/NicWxqLvw6OplS0vPzrK6K3ibYnpjnicoXrmKWXRZicdico2kjqnvSsg36Pt2lYRE3jlLA3FSQXImjhyvo2hRRIf2NbtK1l3jaSgsJp0CeMu6ePg/640?wx_fmt=png&from=appmsg "")  
  
**2.在请求头**  
**Authorization**  
字段中，存在明显的SQL注入攻击行为。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NicWxqLvw6OpM9Ige0JMF964cGuTemdA9XnVC839ugffbCnWe6o5C5dOnOKPUEBCadIbd65JF7ZrtfP3KqoQAAVs04Ihv2HtkdbT4zXqwYUc/640?wx_fmt=png&from=appmsg "")  
  
**3.分析告警日志，****发现攻击者虽然使用**  
**不同代理IP**  
进  
行攻击，但外联下载勒索病毒的  
**C2地址相同。因此根据相同 C2 地址判断：不同IP是同一批攻击者进行的勒索攻击。**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NicWxqLvw6OrgT7ZzZj7feJuwicd38j03riaB4XNAjZFZbLQphWg93jkKrXJzJYWTXArPPGSTnnk20KckYMib6tLia29Ocbj2hicz4ibr3TJwbnR0g/640?wx_fmt=png&from=appmsg "")  
  
**03**  
  
**漏洞分析**  
  
  
**1**  
  
**url中的**  
IsFree=1跳过默认授权校验：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NicWxqLvw6Oqib6JDrNgEvD2iaoop830yn6sjaqEbAUBrpaOW7vXaUSIgkp4wRasic9yLZehqFh8n5I4AqyWa3stF0ngySDiaTQDbgtIEmR8Tiaqg/640?wx_fmt=png&from=appmsg "")  
  
  
**2**  
  
**从请求中获取Authorization**  
字段内容并进行  
base64  
解码  
：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/NicWxqLvw6Oqwp24lxJgCR4Qy9IFTia9A5M7LZ1ZXChL73Qr5BkndgghYYWaKeAiaqS17QUZXY2yH8KCOasBiaMKhV8OGPVp74xzno1czbpVI3M/640?wx_fmt=png&from=appmsg "")  
  
  
**3**  
  
**根据请求中的GetAnonymousTPlusToken**  
方法，解析Authorization 字段内容：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NicWxqLvw6OogAOJjmZqp5PLCPxGVZVm38n1hf2vdxRguY4Z4aBoDPKAPEda338viatmO4d5zArptxdTNUWVZsuibG0A0eJca4rHvqh5ITvRfg/640?wx_fmt=png&from=appmsg "")  
  
  
**4**  
  
**GetAnonymousTPlusToken方法获取orgID**  
 参数后，未做任何处理。最终在方法内部调用  
GetAccountByCloudOrgId  
方法时，触发  
**SQL注入漏洞**  
：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NicWxqLvw6OqjwJbKSgMgoicXUwZSTnRsrBuEmqdKLSJGtGC2c1sna0ZCrhibMNgA6PazRjSrgr5oviacKRoldia2NjloY61ZB7DoNPz9rVzdmyU/640?wx_fmt=png&from=appmsg "")  
  
  
**04**  
  
**漏洞复现**  
  
  
Authorization字段内容编码后进行复现，确认可  
任意执行SQL语句，  
并可  
**执行系统命令**  
：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/NicWxqLvw6Op3ZZVUIxOLHu3OuJEsbFRuFpCrAZTdIuJ2FeP2noymGU2KI3icgtic1f8hQuOSmcuia6fcHMrJ3SzupQrsbKk60Qr551KKj4ib4ro/640?wx_fmt=png&from=appmsg "")  
  
  
**05**  
  
**防范建议**  
  
  
当前勒索病毒攻击形势严峻，  
记住  
"预防 + 备份"  
两个  
核心。建议依托技术手段，持续收缩攻击暴露面，配套管理制度规范，确保全流程安全操作。网络安全建设并非一劳永逸，需紧跟勒索攻击新态势，常态化开展风险评估，持续迭代完善防护体系。  
  
**具体可落地的**  
**五大核心防护动作**  
  
**① 补丁与终端防护**  
  
建立常态化补丁更新机制，第一时间修复已知高危漏洞，部署终端检测与响应系统，实现对异常进程的实时识别与拦截；  
  
②   
网络隔离与访问控制  
  
收紧边界访问策略，关停非必要服务端口，全面梳理互联网暴露面资产，核心系统严格执行IP白名单准入机制；  
  
③数据备份与隔离  
  
遵循"3-2-1"备份原则（保留至少3份数据副本，存储在2种不同介质上，其中1份置于异地），同步部署本地与云端备份副本，每季度开展一次备份完整性校验与恢复演练；  
  
④安全意识培训  
  
定期面向全员开展钓鱼邮件识别、恶意链接处置、陌生介质管控等场景化培训，把人为疏忽的风险降到最低；  
  
⑤安全监测防护  
  
开展7×24小时安全监测常态化工作，异常攻击第一时间拦截，配合标准化应急响应流程，压缩威胁驻留时间。  
  
**中国电信安全依托"预防为主、备份兜底"****防护思路，已成熟落地针对上述勒索病毒全流程防护的**  
**全栈产品与服务****（如：中国电信安全托管服务MSSP高级版、等保助手WAF、等保助手NDR、云镜主机安全、天翼安全大脑防护版等），适配不同规模企业IT架构，补齐勒索防护能力短板。**  
  
服务热线  
  
**400-925-9120**  
  
**敬请垂询**  
  
供稿：运营保障部（MSSP运营团队）  
  
排版：马子豪  
  
编辑：陈师慧  
  
校对：李雪  
  
执行主编：田金英  
  
主编：冯晓冬  
  
  
**推荐阅读**  
  
  
[](https://mp.weixin.qq.com/s?__biz=Mzk2NDIzNTYwOA==&mid=2247490637&idx=1&sn=bec99105c68ff398fe70c204cf67cb36&scene=21#wechat_redirect)  
  
[警惕！Sorry勒索病毒来袭！](https://mp.weixin.qq.com/s?__biz=Mzk2NDIzNTYwOA==&mid=2247490637&idx=1&sn=bec99105c68ff398fe70c204cf67cb36&scene=21#wechat_redirect)  
  
  
  
[](https://mp.weixin.qq.com/s?__biz=Mzk2NDIzNTYwOA==&mid=2247490549&idx=1&sn=c0547b2f3a6923c94add91ee1f200482&scene=21#wechat_redirect)  
  
**“我怎么中毒了？”——MSSP主动服务+AI预警，一次跑赢勒索攻击的防护实践**  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/z7xPqlc0GbBZdrnibkon0HxogO9iazwQy5cqsw6fRdkPujrZZCuVnk7ywgyYA7yTfRIIkEXpdpDfQlkFMENO9P0Q/640?wx_fmt=gif&from=appmsg "")  
  
  
