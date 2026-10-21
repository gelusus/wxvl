#  发生了什么：PHP 出现两个高危漏洞（php2shell）  
原创 messfree
                        messfree  MessFreeSecurity   2026-10-21 06:36  
  
**2026年10月20日，PHP 核心被披露存在两个远程代码执行漏洞（CVE-2026-109618、CVE-2026-109619），CVSS 评分均为 9.2。**  
  
漏洞影响范围较广。相关细节和一段演示视频提前在网上出现，视频中展示了一个名为 php2shell 的工具。  
  
目前所有公开渠道均未见可用的利用 PoC 或稳定的 exp。  
  
PHP 官方计划于 10月21日 14:00 UTC（北京时间 22:00）发布紧急安全更新。补丁尚未正式放出。逆向补丁出 exp 可能在 2 个小时内出来，所以今夜升级是最优解。  
  
**临时缓解措施：**  
- 在 php.ini 中设置 session.upload_progress.enabled = Off  
  
- 或在 Web 服务器层面拦截 multipart/form-data 的 POST 请求  
  
