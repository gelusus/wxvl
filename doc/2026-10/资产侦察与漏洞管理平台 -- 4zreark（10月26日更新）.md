#  资产侦察与漏洞管理平台 -- 4zreark（10月26日更新）  
 Web安全工具库   2026-10-27 01:53  
  
===================================  
  
**免责声明**  
  
请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，作者不为此承担任何责任。工具来自网络，  
安全性自测  
，  
大家都要把工具当做病毒对待，在虚拟机运行。  
如有侵权请联系删除。个人微信：  
ivu123ivu  
  
  
**0x01 工具介绍**  
  
资产侦察与漏洞管理平台：从企业/域名种子出发做资产展开、企业股权穿透、数字资产采集、指纹与服务识别、漏洞复核，并把结果归并成可检索、可下钻的资产图谱。任务：  
```
项目种子
  └─ 巡航计划 / 手工任务
       └─ TaskService.Create
            └─ River 入队（按 task_id 唯一）
                 └─ master 取 job → NodeHub.Dispatch(capability)
                      └─ WebSocket 下发 scan 指令
                           └─ worker 拉任务 → StreamingPipeline
                                └─ 子域 → 接管 → 端口 → 指纹 → 爬虫 → 目录 → 敏感 → 漏洞
                                     └─ 结果/资产/路由回传 → master 落库
```  
  
  
**0x02 安装与使用**  
  
一、环境要求  
```
- Go **1.26.3+**
- Node.js 20+（建议 22）
- PostgreSQL **16**
- Linux 建议安装 `ca-certificates`、`tzdata`
```  
  
二、初始化数据库  
```
createuser zre --pwprompt
createdb -O zre zreark
psql -d zreark -c 'CREATE SCHEMA IF NOT EXISTS zre AUTHORIZATION zre;'
```  
  
网盘下载链接（一定要在虚拟机运行）：  
```
后台回复：20261027
获取下载链接，仅一天有效
```  
  
  
  
  
**·****今 日 推 荐**  
**·**  
<table><tbody><tr><td data-colwidth="287" style="word-break: break-all;"><p><span leaf=""><br/></span></p><p><span leaf=""><img class="rich_pages wxw-img" data-aistatus="1" data-imgfileid="100035404" data-s="300,640" data-src="https://mmbiz.qpic.cn/sz_mmbiz_jpg/U7LDNXUGXQvxMTm2RMVbQWjJu8QKNicspVHqtJ76WbXIFHkXWHGlN2yU6kfKqsU6GCia0ZOkRRp6GXRsF2iaOBvNgz6n7eq1mayJRNEcBwpds0/640?wx_fmt=jpeg&amp;from=appmsg" data-type="jpeg" type="inline"/></span></p></td><td data-colwidth="287" style="word-break: break-all;"><section nodeleaf=""><img class="rich_pages wxw-img" data-aistatus="1" data-imgfileid="100035049" data-ratio="1.2469635627530364" data-s="300,640" data-src="https://mmbiz.qpic.cn/sz_mmbiz_jpg/8H1dCzib3UibsC4yYFwgTnJrN0q57DearHJhaWSE6XQllpkUviaibg5MqTYgdUQYDNt8ysfV2v6o4jsN34pmq3DAOg/640?wx_fmt=jpeg&amp;from=appmsg" data-type="jpeg" data-w="1235" style="letter-spacing: 0.578px;"/></section></td></tr></tbody></table>  
  
