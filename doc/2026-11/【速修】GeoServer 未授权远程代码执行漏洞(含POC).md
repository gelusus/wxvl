#  【速修】GeoServer 未授权远程代码执行漏洞(含POC)  
原创 网络安全透视镜
                    网络安全透视镜  网络安全透视镜   2026-11-24 14:25  
  
2026 年 11 月，开源地理空间数据服务器 GeoServer 被披露存在一条  
   
未认证远程代码执行漏洞链  
   
。该链条由  
   
五个独立 CVE  
   
组成，攻击者不需要任何凭据，就能依次利用默认配置中的缺陷读取服务端安全配置、获取管理员会话，并最终以进程权限执行任意命令。  
  
本文看点  
  
01  
  
五处缺陷如何串联  
  
02  
  
三个版本分支受影响  
  
03  
  
PoC 与修复路径  
  
01  
PRODUCT  
### GeoServer 承载什么数据  
  
  
GeoServer 是一款开源地理空间数据服务器，基于 Java 实现，用于发布  
   
WMS、WFS、WCS 等 OGC 标准服务  
   
。其中 WFS（Web 要素服务）负责  
   
按空间范围与属性条件返回矢量数据  
   
，是业务系统调用地图图层的常用接口。  
  
该产品主要承载  
   
政务、测绘、自然资源  
   
以及行业 GIS 平台的图层与空间数据发布，在政府与行业地理信息基础设施中部署较广。此次漏洞链的入口，正是  
   
默认开放的 WFS 服务  
   
。  
  
  
02  
VULNERABILITY  
### 五处缺陷环环相扣  
  
  
本次风险  
   
并非单一漏洞  
   
。五处缺陷  
   
各有独立编号  
   
，在默认安装状态下可以依次触发：  
  
<table><thead><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">环节</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">缺陷要点</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE 编号</span></section></th></tr></thead><tbody><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">1</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">超长标识触发异常，回显数据目录路径</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE-2026-129244</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">2</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">服务名截断绕过鉴权，匿名可写存储查询</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE-2026-129243</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">3</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">重载二次解析，可读安全配置文件</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE-2026-120705</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">4</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">固定签名密钥，可伪造管理员会话</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE-2026-129245</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">5</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">用户服务反序列化，最终执行命令</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE-2026-129242</span></section></td></tr></tbody></table>  
  
前四处单独存在时  
   
只造成信息泄露或权限绕过  
   
；第五处需要管理员权限才能投递数据，而  
   
第四处恰好提供了伪造的管理员会话  
   
。  
  
从公开 PoC 披露的流程看，完整链条分为四步：  
  
1  
用超长   
storedQuery_id  
 触发 WFS 异常，回显数据目录绝对路径。  
  
2  
以   
service=x/WFS  
 绕过   
wfs.CreateStoredQuery  
 规则，匿名写入带转义   
xi:include  
 元素的存储查询，再通过   
DescribeStoredQueries  
 取回   
security/config.xml  
 与   
users.xml  
。  
  
3  
利用其中的 remember-me 密钥与 admin 口令摘要伪造 Cookie，换取管理员会话。  
  
4  
以管理员身份通过 REST 接口上传 HSQLDB 数据库并注册为 JDBC 用户组服务，在查询用户时触发   
OTHER  
 列反序列化，经   
BeanComparator  
／  
TemplatesImpl  
 链执行命令。  
  
五者在默认配置下相互衔接，  
   
只修补其中一环不足以消除风险  
   
。  
  
  
03  
AFFECTED  
### 三个版本分支受影响  
  
  
受影响的版本包括   
GeoServer 2.27.6 及更早版本  
   
、2.28.0 至 2.28.5、3.0.0 至 3.0.1。修复版本为 2.27.7、2.28.6、3.0.2 及对应分支的更高版本。  
  
<table><thead><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE 编号</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">严重等级</span></section></th><th style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;background: rgb(220, 38, 38);color: rgb(255, 255, 255);font-weight: 700;padding: 8px 10px;text-align: left;"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">评分</span></section></th></tr></thead><tbody><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE-2026-129244</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">中危</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">6.9</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE-2026-129243</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">高危</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">8.8</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE-2026-120705</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">高危</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">8.6（CVSS 3.1）</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE-2026-129245</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">严重</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);background: rgb(254, 242, 242);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">9.2</span></section></td></tr><tr style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;"><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">CVE-2026-129242</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">高危</span></section></td><td style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;padding: 8px 10px;border-bottom: 1px solid rgb(254, 226, 226);color: rgb(55, 65, 81);"><section><span leaf="" style="scrollbar-width: thin !important;scrollbar-color: rgba(255, 255, 255, 0.32) rgb(16, 16, 20) !important;">8.6</span></section></td></tr></tbody></table>  
  
  
  
04  
CONDITIONS  
### 默认配置即可触发  
  
  
利用这条链  
   
需要满足的条件并不多  
   
：  
  
认证要求  
  
无需任何凭据，从匿名 WFS 请求即可开始；针对 GeoServer   
default  
   
文件用户服务中启用的   
admin  
   
用户。  
  
配置要求  
  
默认配置即可触发，不依赖管理员误操作或额外安装扩展。  
  
系统要求  
  
PoC 载荷通过   
/bin/sh -c  
   
执行命令，目标主机须为 Linux；单条命令长度上限为 512 个可打印 ASCII 字符。  
  
权限影响  
  
官方   
docker.osgeo.org/geoserver:3.0.0  
   
镜像以 root 运行 GeoServer，利用后直接获得 root 权限；其他部署方式下，权限取决于运行进程的身份。  
  
  
  
05  
POC  
### 完整利用链已公开  
  
  
漏洞发现者 Crypto-Cat   
已在 GitHub 公开完整利用代码  
   
，仓库地址为 https://github.com/Crypto-Cat/geoserver-unauth-rce-poc 。  
  
发现者于 2026 年 8 月 2 日将该漏洞链报告给 GeoServer，官方于 11 月 23 日发布五份安全公告并完成修复。仓库提供的脚本可完成从匿名 WFS 请求到命令回显的完整流程，示例运行命令为：  
  
.  
.  
.  
bash  
  
python3 geoserver_preauth_rce.py -u http://127.0.0.1:18080/geoserver -c id  
  
在官方 3.0.0 镜像上的运行输出显示，命令执行后  
   
返回 uid=0(root)  
   
，即已获得 root 权限。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/m3tfzlbEQPq5LqbYryyf0Dic3HKibfibWJMSZ0hsnlLFSRj2N5Rgl0aDwY7UWVsAdaIBicEksyNKgbQHeW04sJWDxjuoz5LJE2ZHs3WyibMat1zI/640?wx_fmt=png&from=appmsg "")  
  
  
  
06  
FIX  
### 升级为主，缓解为辅  
  
  
建议  
   
升级至 3.0.2、2.28.6、2.27.7  
   
及对应分支的更高版本。由于该链路涉及多个独立缺陷，  
   
仅更新单个组件不足以消除风险  
   
。官方下载入口为 https://geoserver.org/download/ ，升级前应核对发布说明与下载页面的版本信息，并确认所用扩展与数据目录的兼容性。  
  
在完成升级前，可采取以下临时措施：  
  
1  
限制 WFS 服务的对外暴露，仅允许可信来源访问。  
  
2  
对匿名访问启用统一身份认证与访问控制。  
  
3  
关闭非必要的存储查询功能。  
  
4  
限制管理界面、REST 接口与用户服务的访问来源。  
  
上述措施  
   
只能阻断链路的入口环节  
   
，无法消除配置泄露、会话伪造与反序列化风险，  
   
须与升级同步推进  
   
；相关限制可能影响依赖匿名 WFS 调用的业务，升级完成后应及时移除。  
  
  
∞  
TIMELINE  
### 时间线与参考资料  
  
  
  
2026 年 8 月 2 日  
  
漏洞发现者向 GeoServer 报告该漏洞链。  
  
  
2026 年 11 月 23 日  
  
GeoServer 发布五份安全公告，修复版本同步上线。  
  
  
2026 年 11 月 24 日  
  
长亭安全应急响应中心发布复现通告；PoC 仓库公开可用。  
  
参考资料：  
  
https://cryptocat.me/blog/research/analysis/geoserver_preauth_rce/  
  
https://github.com/Crypto-Cat/geoserver-unauth-rce-poc  
  
https://geoserver.org/download/  
  
END  
  
我是 网络安全透视镜 关注网络安全与漏洞情报。  
  
如果你觉得今天这篇有收获，欢迎  
 **点赞、在看、转发**  
   
三连，我们下篇见。  
  
  
