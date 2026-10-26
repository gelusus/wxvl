#  漏洞预警 | Drupal远程代码执行漏洞  
浅安
                    浅安  浅安安全   2026-10-25 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-108097  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
Drupal是一款基于PHP开发的开源企业级内容管理框架，广泛用于政府、金融、媒体等机构搭建门户网站、业务系统与Web应用。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/NQlfTO30MhzAnDhnrasiaqbbLGOfSchKiauI2l1EJdHLn7dlXcJEVhvdY12QTH6nrUKgMKTFWYSF1yoeFx0j8Vql17iabqoBRWcUSvswbKicibicI/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-108097**  
  
**漏洞类型：**  
远程代码执行  
  
**影响：**  
执行任意命令  
  
**简述：**  
Drupal存在远程代码执行漏洞，由于其FieldUiTable::tablePreRender：Field UI表格渲染时会把row的#region_callback当作PHP callable直接调用，并把整行数组按引用传入。如果目标站点存在一个贡献模块或自定义代码路径，允许HTTP请求数据影响Field UI row的#region_callback，攻击者就可以让Drupal执行任意PHP callable。  
  
**0x04 影响版本**  
- 8.9.0 <= Drupal <= 10.6.18  
  
- 11.0.0 <= Drupal <= 11.3.18  
  
- 11.4.0 <= Drupal <= 11.4.9  
  
- 12.0.0-alpha1 <= Drupal < 12.0.0 beta2  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://new.drupal.org/  
  
  
  
