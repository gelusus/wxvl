#  AI 赋能的漏洞播报系统、获取最新漏洞Poc、自助接入漏洞社区、双向Nuclei通道，持续监控全网漏洞披露信息  
wr0x00
                    wr0x00  渗透安全HackTwo   2026-10-19 16:00  
  
0x01 工具介绍  
  
ai_bobao 是一款轻量 AI 漏洞情报播报系统，多源并发抓取 Seebug、先知、Exploit-DB 等漏洞社区 RSS 资讯，10 分钟轮询采集。支持自助接入新增漏洞源，系统可自动探测 Feed。依托大模型将漏洞信息结构化整理，自动补全漏洞原理、影响范围并生成 Python PoC，英文情报自动中文化。内置双向 Nuclei 通道，既可把漏洞情报导出为 Nuclei v3 模板，也能增量同步官方模板库。文件式存储无需数据库，搭配 Web 控制台与完整 API，7×24 小时持续监测全网漏洞披露，方便安全人员快速获取漏洞情报。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAW6hiaDLtkbvGTXSSLOlso0S7BtrKZToPibicNSueH59ibDZwFNH9AKzbfAdL6iaia2ibSgpFLkYpvtOfP0DdVnWbfKHia6yNFLK14ICia8/640?wx_fmt=png&from=appmsg "")  
  
  
  
  
  
  
  
注意：  
现在只对常读和星标的公众号才展示大图推送，建议大家把  
**渗透安全HackTwo**  
"**设为****星标⭐️**  
"  
**否****则可能就看不到了啦！**  
  
**下载地址在末尾 #渗透安全HackTwo**  
  
0x02   
功能介绍  
  
✨核心特点  
  
- **多源聚合**  
 — 并发抓取 Seebug、先知、Exploit-DB 的 RSS 与正文，单源失败不影响整体，每个请求有硬性超时。  
  
- **自助接入新社区**  
 — 设置页填一个社区地址，系统自动探测其 feed 并由 AI 判断是否为漏洞社区，确认后落配置即抓。  
  
- **AI 补全**  
 — LLM 将文章转为结构化情报：名称、简介、原理、攻击范围；文章已含 PoC 则原样收录，缺失则按公开思路补写 Python PoC。  
  
- **双向 nuclei 通道**  
 — 收录的漏洞可导出为 nuclei v3 模板；上游模板库的新增模板也可增量同步入库。  
  
- **自动中文化**  
 — 英文情报与模板元数据经 LLM 译为中文，CVE 编号、组件名、函数名保留原文。  
  
- **文件式存储**  
 — 无数据库依赖，每条漏洞一个目录，换库只改 store.py  
。  
  
- **内置 Web 控制台**  
 — 漏洞列表、详情、模板库、在线设置，定时后台刷新并展示倒计时。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVdiaydS4KBeFb8rfupXw3Ja5wibRoPvSu2MibkaDXR0CEQ88RB4dmvt4jjmBxmoricJyBiclPnFGHnbMiaI852jVcUH9tiat2czHibbtA/640?wx_fmt=png&from=appmsg "")  
###   
  
  
0x03 更新介绍  
```
内置源与自定义源叠加抓取，自定义源每轮现读配置，改完不用重启。
AI 判活不可用时自动退化为标题关键词匹配，宁可放过也不误杀——把正常社区挡在门外的代价远高于放进一个无关站点。
```  
  
###   
  
0x04 使用介绍  
  
📦安装与使用指南  
  
配置  
```
cd backend
cp config.example.yaml config.yaml
# 编辑 config.yaml 填入 LLM 端点与 API Key（不填则 AI 相关功能不可用）
```  
  
后端（默认端口 8100）  
```
pip install fastapi uvicorn pyyaml feedparser
python main.py
```  
  
前端  
```
cd ../frontend
npm install
npm run dev        # http://localhost:5173
```  
  
打开 http://localhost:5173  
 访问 Web 控制台。后端http://localhost:8100  
  
  
**0x05 内部VIP星球介绍-V1.5（福利）**  
  
          
如果你想学习更多**渗透测试技术/应急溯源/免杀工具/挖洞SRC赚取漏洞赏金/红队打点等**  
欢迎加入我们**内部星球**  
可获得内部工具字典和享受内部资源和  
内部交流群，  
**每天更新1day/0day漏洞刷分上分****(2026POC更新至18223+)**  
**，**  
包含全网一些**付费扫描****工具及内部原创的Burp自动化漏****洞探测插件/漏扫工具等，AI代审工具，最新挖洞技巧、SKILL等**  
。shadon/  
Hunter  
/  
0zone  
/  
Zoomeye  
/Quake/  
Fofa高级会员/AI账号  
/CTFShow等各种账号会员共享。详情点击下方链接了解，觉得价格高的师傅后台回复"   
**星球**  
 "有优惠券名额有限先到先得  
**❗️**  
啥都有  
**❗️**  
全网资源  
最新  
最丰富  
**❗️****（🤙截止目前已有3000+多位师傅选择加入❗️早加入早享受）**  
  
****  
最新漏洞情报分享：  
https://t.zsxq.com/DSAvv  
  
****  
  
**👉****点击了解加入-->>内部VIP知识星球福利介绍V1.5版本-1day/0day漏洞库及内部资源更新**  
  
****  
  
  
结尾  
  
# 免责声明  
  
  
# 获取方法  
  
  
**公众号回复20261020获取下载、回复 加群 获取交流群**  
  
****  
  
# 最后必看-免责声明  
  
  
      
文章中的案例或工具仅面向合法授权的企业安全建设行为，如您需要测试内容的可用性，请自行搭建靶机环境，勿用于非法行为。如  
用于其他用途，由使用者承担全部法律及连带责任，与作者和本公众号无关。  
本项目所有收录的poc均为漏洞的理论判断，不存在漏洞利用过程，不会对目标发起真实攻击和漏洞利用。文中所涉及的技术、思路和工具仅供以安全为目的的学习交流使用。  
如您在使用本工具或阅读文章的过程中存在任何非法行为，您需自行承担相应后果，我们将不承担任何法律及连带责任。本工具或文章或来源于网络，若有侵权请联系作者删除，请在24小时内删除，请勿用于商业行为，自行查验是否具有后门，切勿相信软件内的广告！  
  
  
  
# 往期推荐  
  
  
**1.内部VIP知识星球福利介绍V1.5（AI自动化）**  
  
**2.CS4.8-CobaltStrike4.8汉化+插件版**  
  
**3.全新升级BurpSuite2026.4专业(稳定版)**  
  
**4. 最新xray1.9.11高级版下载**  
  
**5. 最新HCL AppScan Standard**  
  
  
渗透安全HackTwo  
  
微信号：关注公众号获取  
  
后台回复星球加入：  
知识星球  
  
扫码关注 了解更多  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/RjOvISzUFq6qFFAxdkV2tgPPqL76yNTw38UJ9vr5QJQE48ff1I4Gichw7adAcHQx8ePBPmwvouAhs4ArJFVdKkw/640?wx_fmt=png "二维码")  
  
  
  
  
上一篇文章：  
[Nacos配置文件攻防总结|揭秘Nacos被低估的攻击面](https://mp.weixin.qq.com/s?__biz=Mzg3ODE2MjkxMQ==&mid=2247492839&idx=1&sn=b6f091114fbd8e8922153a996c8f4f1c&scene=21#wechat_redirect)  
  
  
