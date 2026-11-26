#  漏洞复现｜GeoServer 未授权 RCE！ 默认配置直接 root！  
 Poc复现   2026-11-26 03:59  
  
   
  
# 免责声明  
  
「此公众号所分享的网络安全知识、信息及工具皆来源互联网公开收集整理，仅供学习和研究使用，请勿用于非法测试活动。由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，产生的一切后果由读者自行承担，皆与本公众号及文章作者无关、不负任何责任！！！」「如有内容侵权，请联系我们删除处理。谢谢合作！！！」  
# 漏洞概述  
  
CVE-2026-129244、CVE-2026-129243、CVE-2026-120705、CVE-2026-129245 与 CVE-2026-129242分别是信息泄漏、鉴权绕过、任意文件读取、会话伪造和反序列化；组合起来，却是一条完整的预认证 RCE 链——攻击者从一条匿名的 WFS 请求出发，不需要任何账号口令，最终在服务器上以 root 身份执行任意命令。  
# 影响范围  
  
受影响版本：GeoServer ≤ 2.27.6；2.28.0 – 2.28.5；3.0.0 – 3.0.1  
修复版本：2.27.7 / 2.28.6 / 3.0.2。  
# 环境搭建  
### docker  
```
docker pull docker.osgeo.org/geoserver:3.0.0docker run -d --name geoserver-lab -p 18080:8080 docker.osgeo.org/geoserver:3.0.0
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qvo4lzibsSczg7t5sNw10aeQ1sibOqwlOeJicZXZuBUECPmeytXQfYZm9ozgax3bPu4lE6cBGwhk2azc3pgBrxkP06CgjRia6yFc1YuKoSfWrOw/640?wx_fmt=png&from=appmsg "")  
# 空间测绘  
### Fofa  
```
title="geoserver" || app="geoserver"
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qvo4lzibsScztblrmLucwJcRT9e9hQmoxZz72ILib2W0DfEX9hmJZ3SInMicOJR1IM8lBnSl4RW8UvSftGdu3VzK3xEvvwYX5iaE4p6jK2LI208/640?wx_fmt=png&from=appmsg "")  
# 漏洞自查  
#### Nuclei  
```
nuclei.exe -u 目标URL -stats -silent -t geoserver-CVE-2026-129244-wfs-xinclude-preauth-fileread.yaml  
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qvo4lzibsScxx76y2vv7b0yiccz4FVYaziaqFiagia4e6mTggbcWLDvq4nLvVCfz0mF3MWqwaMaiaVYU5JeVXhC1JlDropTE8RuMHa89eHVZwbIw4/640?wx_fmt=png&from=appmsg "")  
#### Py脚本  
```
python3 geoserver_preauth_rce.py -u http://127.0.0.1:18080/geoserver -c id
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Qvo4lzibsScxExgkicLReCZEwlaturPI8DxBcXiayN1bXLXMCyShL0JQkibsyAEC0B6KfDdHcwN6S1TvwAuTIyXgiaWicEZaohib3tLqS3sqOIJUoQ/640?wx_fmt=png&from=appmsg "")  
##### 脚本地址：https://github.com/Crypto-Cat/geoserver-unauth-rce-poc/blob/main/geoserver_preauth_rce.py  
# 修复建议  
  
- • 建议联系厂商打补丁或升级版本。  
  
- • 增加Web应用防火墙防护。  
  
  
### poc免费获取  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Qvo4lzibsScw3G1A0IJ0ibmgaebttNpssa5fG0UBG0aolYwMsic4icQnIJtaHrAFbmtdiaw7iawOhcFtNRZicHmAcia17nL2icIJlQSItUic5PK4aI0e8/640?wx_fmt=jpeg&from=appmsg "")  
  
  
   
  
  
