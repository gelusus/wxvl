#  高危 PHP 漏洞 PHP2Shell 来袭｜php2shell-scanner 自查工具使用指南  
原创 维度攻防
                        维度攻防  维度攻防   2026-11-13 04:53  
  
近期 PHP 爆出 PHP2Shell 系列高危漏洞（CVE-2026-109618 / CVE-2026-109619），可实现无鉴权远程代码执行。本文介绍开源自查工具 php2shell-scanner，教运维、安全人员快速检测自有业务是否受影响，附原理、用法、风险说明与修复方案。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/56vojF9ZLWF8SvvNmyTiaKQmpoAgkKP4ZrHRicHSSVibC9L1BmvjsDoIQVpl6sLRh7kUyEN3rKEich1M1TyXf3q7ah8VGMITAeG7aMf1szlZMRk/640?wx_fmt=png&from=appmsg "")  
  
大家好，最近安全圈都在关注 PHP2Shell 这个高危漏洞，很多基于 PHP 搭建的网站、商城、WordPress、Drupal 业务都面临风险。漏洞核心危害：攻击者构造特殊 HTTP 请求，利用 PHP 内存损坏漏洞，在未登录、无上传点的情况下，直接拿下服务器权限，拿到 WebShell，也就是大家常说的 RCE 远程代码执行漏洞。  
  
很多管理员问：我怎么快速判断自己的服务器有没有中招？今天给大家介绍由 Searchlight Cyber 放出的开源自查工具：  
php2shell-scanner。  
> 一句话定位：php2shell-scanner 是面向防御方的探测工具，用来检测目标 PHP 环境是否存在 PHP2Shell 漏洞，  
默认不执行漏洞利用、不拿 shell，属于风险评估工具，而非攻击 POC。  
  
## 一、漏洞简单回顾：什么是 PHP2Shell  
  
PHP2Shell 是一组 PHP 内核内存安全漏洞，对应两个 CVE 编号：✅ CVE-2026-109618✅ CVE-2026-109619  
  
漏洞触发依赖 PHP-FPM 处理特殊的 multipart 表单请求，恶意构造数组嵌套键名，触发内存破坏，最终实现远程代码执行。特点：  
1. 无需账号密码，无鉴权；  
  
1. 影响范围广，大量 PHP 版本受波及；  
  
1. 攻击流量隐蔽，日志很难直接识别；  
  
1. 一旦被利用，攻击者直接接管站点，上传木马、拖库、勒索。  
  
> 不是所有 PHP 站点都一定可利用，受 Web 服务器、Nginx/Apache 配置、PHP 版本、参数限制影响，存在一定环境依赖。  
  
## 二、php2shell-scanner 工具介绍  
  
php2shell-scanner 是命令行开源扫描器，专门针对 PHP2Shell 漏洞做探测，适合运维、安全工程师对内网资产、自有公网站点批量自查。  
### 探测逻辑（三步检测）  
1. 优先触发 PHP 内存耗尽异常，观察服务响应；  
  
1. 其次识别差异化 500 报错，判断 PHP-FPM 进程异常退出；  
  
1. 兜底探测 PHP 版本信息，做版本粗筛。  
  
⚠️ 重点风险提示：部分探测报文会触发 PHP 内存溢出，  
生产业务扫描有概率造成 PHP-FPM 进程崩溃、网站短暂 500，生产环境务必先在测试环境验证，低峰期操作，提前告知业务方。  
  
工具支持：  
- 单个域名 / IP 扫描  
  
- 批量 IP 列表导入扫描  
  
- 输出结果：正常 / 疑似脆弱 / 确认脆弱  
  
- 输出 HTTP 请求与响应详情，方便安全人员复盘  
  
> 区分：这个工具≠webshell 扫描器，它  
不是扫一句话木马，是检测 PHP 底层这个 RCE 漏洞本身，很多人容易搞混。  
  
## 三、工具基础使用（仅供自有资产测试）  
> 环境：Go 编译二进制 / 源码编译，跨平台 Linux、Windows 都支持  
  
  
项目地址：   
https://github.com/assetnote/php2shell-scanner  
  
基础命令示例：  
```
# 扫描单个站点
./php2shell-scanner -u shturl.cc/Xf3i866m1N
# 批量扫描，读取目标列表文件
./php2shell-scanner -f targets.txt
# 输出详细HTTP报文
./php2shell-scanner -u shturl.cc/Xf3i866m1N -v
```  
  
果三种状态：  
  
✅  
Not Vulnerable：未检测到 PHP2Shell 风险  
  
⚠️  
Needs Verification：版本匹配，但探测特征不明显，建议人工复核  
  
❌  
Vulnerable：探测特征命中，确认存在漏洞风险，立刻下线修复  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/56vojF9ZLWHvcKVic0piaQZmicJmichJsNta0gdffaibL8wnsPXZkIfoRDlwpkWjJkhBXnRRu45SYribvHticbXicFvfFNEcS9DWJ92z9qaSOA5R0jg/640?wx_fmt=png&from=appmsg "")  
### 扫描注意事项（必看）  
1. 只扫自己拥有、书面授权的资产，外网随便扫别人网站会触犯《网络安全法》；  
  
1. 生产优先版本自查，再选择性主动探测；  
  
1. 扫描前做好备份，避免 PHP 进程异常导致业务中断；  
  
1. 防火墙、WAF 可能拦截探测包，造成误报 “无风险”，WAF 放行后复测；  
  
1. 扫描结果只做风险参考，不能 100% 替代人工渗透测试。  
  
