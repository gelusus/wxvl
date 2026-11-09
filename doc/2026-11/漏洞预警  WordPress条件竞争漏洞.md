#  漏洞预警 | WordPress条件竞争漏洞  
浅安
                    浅安  浅安安全   2026-11-08 23:50  
  
**0x00 漏洞编号**  
- # CVE-2026-119977  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
WordPress是一款使用PHP语言开发的开源内容管理系统，提供了丰富的主题和插件生态系统，支持博客、企业官网、电商平台等多种场景。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/NQlfTO30MhydhWlf0jP0jzUhEwjF9E0zjT95dGhSI6VpNExsglt6bzHiaqA99LjAQDRMqQ6tVxLP4aWHm7DcoibfzLW2VkkgqsTyS9HQPrpzQ/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0 "")  
  
**0x03 漏洞详情**  
  
**CVE-2026-119977**  
  
**漏洞类型：**  
条件竞争  
  
**影响：**  
执行  
任意代码  
  
**简述：**  
WordPress存在条件竞争漏洞，由于完整包核心升级过程中，升级程序将更新包解压至wp-content/upgrade/下基于包名生成的可预测、Web可访问临时目录。临时目录包含安装程序setup-config.php，且在升级窗口期间可被外部HTTP请求直接执行。攻击者可利用该漏洞，在升级窗口内通过竞争条件注入恶意配置，最终实现任意代码执行，导致服务器完全沦陷。  
  
**0x04 影响版本**  
- 4.7.0 <= WordPress <= 4.7.38  
  
- 4.8.0 <= WordPress <= 4.8.33  
  
- 4.9.0 <= WordPress <= 4.9.34  
  
- 5.0.0 <= WordPress <= 5.0.30  
  
- 5.1.0 <= WordPress <= 5.1.27  
  
- 5.2.0 <= WordPress <= 5.2.29  
  
- 5.3.0 <= WordPress <= 5.3.26  
  
- 5.4.0 <= WordPress <= 5.4.24  
  
- 5.5.0 <= WordPress <= 5.5.23  
  
- 5.6.0 <= WordPress <= 5.6.22  
  
- 5.7.0 <= WordPress <= 5.7.20  
  
- 5.8.0 <= WordPress <= 5.8.18  
  
- 5.9.0 <= WordPress <= 5.9.19  
  
- 6.0.0 <= WordPress <= 6.0.17  
  
- 6.1.0 <= WordPress <= 6.1.15  
  
- 6.2.0 <= WordPress <= 6.2.14  
  
- 6.3.0 <= WordPress <= 6.3.13  
  
- 6.4.0 <= WordPress <= 6.4.13  
  
- 6.5.0 <= WordPress <= 6.5.13  
  
- 6.6.0 <= WordPress <= 6.6.10  
  
- 6.7.0 <= WordPress <= 6.7.10  
  
- 6.8.0 <= WordPress <= 6.8.11  
  
- 6.9.0 <= WordPress <= 6.9.10  
  
- 7.0.0 <= WordPress <= 7.0.7  
  
- 7.1.0 <= WordPress <= 7.1.3  
  
**0x05****POC状态**  
- 已公开  
  
**0x06****修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本****：**  
  
https://cn.wordpress.org/  
  
  
  
  
