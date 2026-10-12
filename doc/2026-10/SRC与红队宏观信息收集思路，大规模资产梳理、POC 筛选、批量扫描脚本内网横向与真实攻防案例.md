#  SRC与红队宏观信息收集思路，大规模资产梳理、POC 筛选、批量扫描脚本内网横向与真实攻防案例  
1141939898653671
                    1141939898653671  渗透安全HackTwo   2026-10-12 00:24  
  
**0x01 简介**  
  
本文对比 SRC 与护网红队的信息收集差异，护网场景讲究宏观批量挖掘资产，在多目标抢分的压力下，优先寻找薄弱资产快速获取权限。详解 ENScan、无影、Nuclei、Afrog、DDDD 工具链落地，包含控股资产探测、资产清洗、全端口扫描，附带 POC 筛选、批量扫描脚本实操。同时介绍供应链攻击、Host 碰撞等拓展手段，结合 Druid、帆软、小程序越权、内网横向等真实攻防案例，总结打点技巧与实战踩坑经验。  
  
  
  
  
  
  
  
  
> 本文内容仅用于网络安全技术学习与交流，所有操作均在授权测试环境下完成。严禁将文中技术用于未授权检测与非法攻击，违者责任自负。  
  
  
现在只对常读和星标的公众号才展示大图推送，建议大家把**渗透安全HackTwo“设为星标”，否则可能就看不到了啦！**  
  
参考文章  
：  
```
https://xz.aliyun.com/news/90794
https://www.hacktwohub.com/
```  
  
**末尾可领取挖洞资料/加圈子 #渗透安全HackTwo**  
  
**0x02 正文详情**  
  
## 目标分配  
  
基本到现场裁判组织一下赛前宣讲,不要对企业数据造成破坏等等....然后队长开始抽取私有目标名单和第一天就公开的靶标单位名单,靶标单位拿下就有大量的分数加成,有头铁的师傅会选择第一天就着重对靶标单位进行细致的信息收集,死磕一个入口  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAV4HJwgRWEk0ZgkAzHLwjA3aq6PWXnlkkLVG03ruMTKQciazibxHADS9Yt4gKLu8z9aus1I743OUjHXH0J3MBpbE2BU6HztickUEk/640?wx_fmt=png&from=appmsg "")  
  
  
有些攻防资产会提前向参演防守单位企业收集好,防守方上报到表格,主域名、子域名、  
IP  
、有些还会提供  
C  
段划分,但是如果一开始就盲目的扫  
C  
段 很容易打偏资产所以一般来说是前期没有产出才选择的手段  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAV8ibGVlUxPs2CiavMUeF7toQbu9BP8xJE94oQDNjx8uyT1tfbMCo3SW1ibt3IeaCu05IEH4Of5mB6YSialC5YRaKXVVTulR9y36e8/640?wx_fmt=png&from=appmsg "")  
## 备案号 / 单位名 探测主域名  
  
在目标单位数量庞大的情况的,这款工具无疑是最好的选择关键还是免费使用,配置对应平台  
Cookie  
就可使用,还可以收集下属控股单位资产  
```
//  -type aqc爱企查  -fieid限定了导出的字段位icp列表
enscan-v2.0.0.exe  -f  资产.txt  -type aqc  -field icp
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXUGEMibib6c9hsNo8wYCbclEgleCfDE1Ouer2yF3xnV00hs5C6lxvrPOJqgqK6OpXnWg8OCcFLQAl8ocB0mSIhvYtqWmUmriakoM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXETPpJVv3ZgjsqEUsgzlXTtbEBnb9glEqJAj9BpyXf6oSjOHW22QkhjNlha9YrP18fEOiaJqUYvZhzR5WC3V0B0D3ScT0tpqMs/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUgXxWrd0YicMbiaicmcDbUiaEZ4h3bAYGqj6yNCeBOF0sOEr26DaWMFBsL1jIxqH812pibickcOFEL2HWwraWXuS9JmAym31ta1Ht9A/640?wx_fmt=png&from=appmsg "")  
  
大部分单位控股  
100%  
 企业漏洞都是算作分数的,当企业本身无漏洞可以利用测语法寻找控股资产  
```
//  搜索资产单位制控股子公司占比在100%范围
enscan-v2.0.0.exe -f 资产.txt -type aqc -field icp  -invest 100
```  
  
如果遇到  
AQC  
手动去浏览器验证,不行就退掉账号重新登录,把新  
Cookie  
弄到配置文件里去,单位过多后面会卡死限制访问,切换  
WIFI  
改出口  
IP  
可以解决这个问题  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVlbiaVJItgX6GiaohwfSe10sLD96Kwvdv2KlCeAuT4MSn8ySW6oZCqibic0VXFy4ico2JgzpyeIZ4P1tPl86mpDibiaTqicF6uJXPKvrs/640?wx_fmt=png&from=appmsg "")  
### 无影  
  
无影支持批量查询，配置  
Cookie  
一键查询调用爱站网、站长之家搜索,目前都没有出现被检测情况  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAUWS5aOSScqdEPS4vtJbksuTcoabL1akI899z5c8RrXlcZs9YtHD2nrGLL9eYW9NWy7eHqmKbCkN7Zb1uaE3ChQLuJDZ1cuaA0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXSFjrGrNXn5veBTYAEdTFbSTtX4pTTdToxdndibdibDeV0A0jBveicdIt6AFLLHrYqianwS3FC3OqEfWDaiaK51jTXoK6tuuPvfFLs/640?wx_fmt=png&from=appmsg "")  
  
**资产清洗**  
  
如果已经提供给我们资产的  
xls  
表格的那更好不过,一定程度上节省了时间,但是给予的资产什么都有,区分资产需要一定时间,无影的这项功能可以很好的清洗资产,主域名直接去跑  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXQyDic3PD2Kj9bZLciaeKialCFr8LIMrQhY8hm6uFR25oDBVyicjlYjVcoqkzIvnIDy6bs3xBGsMpgxwEh8AuKAOTrUgHIwkg3ibj4/640?wx_fmt=png&from=appmsg "")  
### Fine  
  
可视化信息收集,功能比较多但最让我喜欢的还是批量  
ICPname  
 批量功能并且也支持设定控股范围,早期此功能是免费的非常香,后面作者也要恰饭就开始收费,感兴趣朋友可以去了解  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUI4gECGibIYOTU9WTPgt2mPE8Fd3lyvgv018MopQFweWwNPXFKe5KIVyNgXgzwp9yvXFp4CyYouQFwxjicTJP7orEdODcEzTHqE/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXTvDxMriaXxkS0gQvExJpt47GXU2QJ9EdULz9zAL4HHQFy9ia9Yd8jLwSDW6WqrguVxDkMNwUicjQEWicX9UYiaGeKcuZ9aQIZ8vGM/640?wx_fmt=png&from=appmsg "")  
  
## 子域名 / IP  
### 无影全测绘  
  
依靠前期备案单位得到有效主域名和IP段开始进行子域名搜集还有全端口扫描,还是依靠无影,网络测绘最常用的也就是  
fofa  
hunter  
360  
 但由于引擎不同那么资产肯定会有偏差,有的资产在另一个测绘可以找到,别的测绘却找不到,最完全的是可以集中查询,无影正是实现了这点,语法已经适配,配置好  
api  
导出后就是全量的资产  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUFdOcibLooFO17HbsDGzvJPPAorDJbniapZibEciaRhiaenXgMGYSicC5oW7fAiapzvQA6G1AB0pjxNMnzOPpYtV4ibt6EqD3VI1SZJ3U/640?wx_fmt=png&from=appmsg "")  
  
由于资产肯定会有重复的情况所以需在  
xls  
表格内对  
ur  
和  
ip  
进行去重这样就是相对完整的测绘收集过程了,针对子域名还可以进行一下存活探测  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWx25B2YpuBr5zB5ib4TVBw1F6otD4asI10OZwfJlSMYP3Ju4x3CdlEBhysLicWUkyZ7bSd5fqGlI7lrcicURqCfIicnZgOe2Hf3xw/640?wx_fmt=png&from=appmsg "")  
### 全端口扫描  
  
IP  
地址是一定需要进行全端口扫描的.特别是在高端口极其容易发现一些隐藏资产服务,同样利用无影进行扫描,支持  
POC  
探测、服弱口令爆破和全量指纹识别,虽然无影可以联动  
Nuclei  
afrog  
等进行扫描但建议另外工具窗口进行,全开有时候会出现卡退情况  
  
  
全端口扫描出的  
IP:port  
资产批量导出,将有效可以访问的资产再使用漏扫过一遍,尽可能覆盖到全部的单位资产面  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXGEiamg3dlAIcnayDK6Kibw65r9lu3Xcgtf3iapfcLicgLj2GAiaOFwFIrMCIFTibYml7tVVMZpZibbo4mdNyzibjbU19OQYzT0oeyj44/640?wx_fmt=png&from=appmsg "")  
  
端口资产在漏洞扫描时就可以再多看看无影的指纹识别有没有什么弱口令的框架,可以只看高危的指纹这为我们检索信息提供了极大的便利,   
Rabbitdruid  
zabbix  
MQTT  
等,口令在公网都可以收集到但也可以总结一份针对框架的小字典,只针对于各种框架弱口令进行爆破,没有框架的站就是正常小字典爆破,师傅们可以自己收集总结  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWh1BIvTL0LyWibVXrnALUEI7iaHSibh1cAGibR7yV8FGhF1AcspV2x2oqXfuu5cuxs7ZYlpVf3RLCfE3CC1UYt3YMuPvcdp7tecIY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAVFhjWibfpq6riaIdVdicVbhib30HawvrHktFIibnVWRMziaUibzfo158REQkNEmYXBgqrBoqPSvyspTsHkLibTbQERnzhDtw4ARhyfYibM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAW9Z2ff5pTT6LbGzpKhibEdiaA0ia3MFGfbobn25DiabicXhbGqhibzY33en6fibjvnoX2iarEkhRcPvrQU9KjseLFkXhHx2R8v8ibCG65c/640?wx_fmt=png&from=appmsg "")  
  
自带服务的弱口令爆破,所以没事可以看一看,捡点漏 捡到一台  
frp ssh  
也是赚的  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXobLQctZ5OX1DWN8qD242ic9MwkU8Bt8s2EePCsgFESuP1eTB3EkNJC0fASaFDQwrUPOmgn18yMOPBFvicSP7dsytYaBVM5Lyqk/640?wx_fmt=png&from=appmsg "")  
  
不止弱口令的服务框架,如果找到列目录也要点进去看看,某次攻防也是因为多看了一眼从而捡到一个大洞  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXClia31Gt1CFThO8SI0PbPiaBLibb0vc1VMNP88Ifm5JYtjudaDUAIicefn8IprB1uDVSN7CLeiaNVsiab5vIibZD3WALorq7Xvtyuc4/640?wx_fmt=png&from=appmsg "")  
  
文件名  
18  
 数字有些还有  
X  
 妥妥以身份证号作为文件名,图片信息则是熟悉的大头照,依旧她奶,贴了表情包保护隐私。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAVeaZOiaBQ3x7pjBKyfTPU5GM2I3iaj7u2DraZR2Ox1gvuxbb0Sic4iaNkVa5FahGK6CyW0ZO84PsXoekJ85kyzZqDrj5pOFVHhAfk/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVk2R9B7eXqUCbAMy4HlDfOhcbJIChK0peHC0KXWXYeoTz4jQohqtP2AnlrYJySMnbhKzOGHYiaVpVwrZYEquB8n5B8FK72jicibI/640?wx_fmt=png&from=appmsg "")  
  
复制全文统计了一下泄露了  
14w  
 ,并且无影还会帮我们扫描到一部分的路径,捡到  
druid  
和  
swagger  
等漏洞,所以针对  
IP  
扫描全端口是非常有必要的   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUyK3XUDgyAOEINWLyKwsBxibZt7o63CMaJuXNPKzcC1mY6ChGib1R0caYuvg3icV31ed1A9dVmfG96rJReASsibKiahEfvicOs9PuK4/640?wx_fmt=png&from=appmsg "")  
### TIPS  
  
上面是比较宏观的收集方式,但如果常规收集并没有发现有效的漏洞,还可以进一步拓展思路发现更多资产,笔者因为并没有实践过所以无法给出实战案例,但收集了一部分其他优秀师傅分享的技术思路,  
  
**C端**  
  
参考文章：  
[聊一聊红队打点那些事](https://mp.weixin.qq.com/s?__biz=MzAxNzkyOTgxMw==&mid=2247492438&idx=1&sn=b48342480dffa2e9c50d91177b9f66c4&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXm2ctORHXK1kiboMMoLZAx5sibiaVgSVH31wV53AJyZJIHSH8r4LPqHUsrAlA9BrnwWSia7VBfQ0gUUQSGLElXS5dyqfZTzFicve9Y/640?wx_fmt=png&from=appmsg "")  
  
**供应链**  
  
供应链攻击就是找该网站的开发商,从开发商下找到其他, 比如目标网站部署了  
xxx  
系统正常手法漏洞不能打入后台,但是发现了也使用这个  
xxx系  
统的网站  
B  
， 找到  
B  
这个站的弱点,尝试拿下源码审计亦或是直开发商,先拿下目标开发商，借助开发商进入目标内网，一般在一些乙方供应商官网都会写下自己的客户单位，借助这点就能定位到这个供应商是否是目标的供应商,但存在辛苦半天打下供应商网站却没有路口可以进入真正的目标内网,所以运气也是一部分  
  
参考文章：  
```
https://zhuanlan.zhihu.com/p/568950328
https://9bie.org/index.php/archives/982/
```  
  
host  
碰撞  
  
资产访问  
IP  
回显  
4xx  
错误，数据包指定host头为某个域名访问,该IP回显正常时，保留  
IP  
可进行碰撞,此技巧SRC同样适用,但是攻防中推荐是限定测试某几家公司,不然范围太大容易不知道打到哪里去了  
```
https://mp.weixin.qq.com/s/v3M1W4oUXqb9j6rn-1XeFA
https://xz.aliyun.com/news/16167
```  
## 漏洞扫描  
### Nuclei / Afrog  
```
nuclei`和`afrog`也是比较知名的漏扫工具了,前者网上有师傅分享过`10wpoc` 我也是配置了一下看看效果,`poc`是很全面,但暴力也是真暴力,运行起来流量特别大简直是小型`ddos`,普通企业网站很容易被打宕机,所以不建议配置这么多的`poc
扫描单个网站: nuclei -u https:baidu.com
扫描列表 :  nuclei -list urls.txt
nuclei -update  # 更新版本
nuclei -update-templates # 更新模版
nuclei -tl  # 列出所有可用模板
nuclei -tl | find /c /v ""  # 列出模板总数
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAW3OoEm4fN83nDTFjpz0qoichVopMvR8z4qicx2oL7ODD1LDUZP2icc1LCrE03FpWXy9QL0AiczQd43GQE3LtNZfkibOX5avsTkWYwo/640?wx_fmt=png&from=appmsg "")  
  
tisp:该工具已经绝版，有需要可以在内部知识星球获取，目前也在整理nucleiPOC均可星球获取更新  
  
在内置  
poc  
基础上可以 增加  
24  
年25年护网报出的  
poc  
,和筛选一部分高危  
poc  
, 把想要添加  
POC  
就在  
C  
盘的文件的配置文件内就可以被工具识别  
```
C:\用户\用户名\nuclei-templates
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXyHcMrL0z0qEF2QicnDZfSxShZoVGCRia0GueIvvYicXZMeT5jFupBM5YBicfrg4wQnLL8539siadEX07uusNKp2j7ic30OsicHB3iaaQ/640?wx_fmt=png&from=appmsg "")  
  
使用  
PowerShell  
面板筛选出严重的漏洞模版然后再存放某一文件夹内,再另外放到  
nuclei加  
载路径下  
```
Get-ChildItem -Path "B:\Penetration\漏扫\nuclei\nucliePOC\nuclei-templates-diy\cve" -Recurse -Filter "*.yaml" | Where-Object {
    Get-Content $_.FullName | Select-String -Pattern "severity: high " -Quiet
} | Copy-Item -Destination "B:\Penetration\漏扫\nuclei\cve-hign-poc"
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAU0y9V2XoFcFCTiaWvaUjJgQt4GeYCdNllK0ERewLbYvOBlib4Bfjrl4faabOPRebia7p4bThDwhI0vZMzM6tHrJ9UJeSdw9urHwM/640?wx_fmt=png&from=appmsg "")  
  
把筛选到的高危  
POC  
先一次放置到  
C  
盘文件夹内,再清理重复  
POC  
 保证效率  
```
$pocPath = "C:\Users\月\nuclei-templates"; if (-not (Test-Path-Path$pocPath)) { Write-Error"模板文件夹不存在: $pocPath"; exit1 }; Get-ChildItem-Path$pocPath-Recurse-Filter"*.yaml"-ErrorAction SilentlyContinue | Get-FileHash-Algorithm MD5 | Group-Object-Property Hash | Where-Object { $_.Count -gt1 } | ForEach-Object { $files = $_.Group.Path; Write-Host"`n发现重复模板组（共$($files.Count)个）："; for ($i = 0; $i-lt$files.Count; $i++) { if ($i-eq0) { Write-Host"  保留：$($files[$i])" } else { Remove-Item$files[$i] -Force; Write-Host"  删除：$($files[$i])" } } }; Write-Host "`n重复POC清理完成！"
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAW3M8hricudWN6mV5buBZFqqt2ZwxIr7rSHBQyDpR52EySNKTOich9hOOibicuibeVmlktkQGudFib4NcR554FqD0QWhFtsGBR8xuZns/640?wx_fmt=png&from=appmsg "")  
```
afrog`小青蛙自身就内置了`1600+poc` 而且也都是比较符合国情的,同样可以添加内置`POC`在其`C`盘位置,目前我都这两款工具都会使用,添加的`POC`都是护网出现的漏洞和高危`POC
列出全部POC  ./afrog.exe -pl  
单个扫描:   ./afrog.exe -t https://example.com
批量扫描： afrog.exe -T url.txt
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAWxaD2csYBOIXgl0jxqRSF2ly8rz26glcB1NRE1JtibwBYI3iaQ4zQooHkQnqJia6DFY6nw4E33OEABILcMlQI0AFqYmwhjJYznO4/640?wx_fmt=png&from=appmsg "")  
### dddd  
  
今年比较火爆的  
dddd  
打点工具,付费版确实是有些东西二开后一直在维护纳入新的  
poc  
;攻防也是利用此工具拿下过入口的,并且指纹识别功能也是不错,省去了我另外对子域名指纹识别的步骤;但是当时没有购买,也是队友朋友帮忙扫的,如果圈子大也是可以让购买的朋友帮忙扫资产,然后导出  
html  
文件即可  
  
tips:二开版可以在内部星球获取  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAUBxnleTTcLXPnUXHpMhOYmrGjmJJfajPNnde7hozTdWhjRFBVwuKHLAQJt1rxKHHzcrcsO6us7S1ZwADKUcFENfNibqUaXiaYqc/640?wx_fmt=png&from=appmsg "")  
  
工具可以配置测绘工具的  
key  
 但只有  
hunter  
可以支持  
icp_name  
语法,另外两个测绘需要升级会员后才可以使用这个语法,所以也就衍生出了攻防演练利用此工具快速抢分方法,开局大量公司名,配置好  
hunter api  
就可以不先收集子域名先用打一波,方法可以看看此师傅文章  
  
[攻防演练中如何快速抢分](https://mp.weixin.qq.com/s?__biz=Mzk0NDU5NTc4OA==&mid=2247484689&idx=1&sn=0022cf3335ff26eb9239077cc3ba3eac&scene=21#wechat_redirect)  
  
```
dddd64.exe -t icp.txt -hunter -sd -p 1-65535 -st syn -sst 150000 -o save.txt
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXYKhjSQCDeBiaxdPIFl0LqEib3h3Ml1JFKj2tjgaMm96xicgLua8rRmPJqo6WErnHZ7iaGP4yiaFnMk54qX3yq96ZRibNlnwdFXPUHU/640?wx_fmt=png&from=appmsg "")  
  
icp  
扫描在分配目标开局就先挂着,等做完  
2、3  
步后再把无影测绘到的所有子域名+  
IP  
再扫一波,并且是  
nuclei  
afrof  
dddd  
工具一起扫  
```
dddd64.exe -t ur.txt -sd -p 1-65535 -st syn -sst 150000
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAVGDxWqO4iaZqk2ERCWMIACkmgtiaH99LMQfLguVkc51R1hbFQ0ibmTTCQL8zCc84ntJmIcmYL9Ip1vSwtsECMzr9Qc1A8pzoN7ZU/640?wx_fmt=png&from=appmsg "")  
  
为了方便我一般写个  
bat  
执行,配置好3个工具的位置)_,指定我的子域名  
txt  
文件直接执行,脚本小子是这样的，没有  
0day  
只能就尽量在打点这块多点奇奇怪怪的技巧，多挑选整合点好用的工具技巧  
```
@echo off
chcp 65001 > nul
title 漏扫工具启动器
color 0a
:: 设置路径变量 - 修改为 ur.txt 以匹配你的实际文件名
set "AFROG_PATH=B:\Penetration\漏扫\afrog\afrog.exe"
set "NUCLEI_PATH=B:\Penetration\漏扫\nuclei\nuclei.exe"
set "DDDD_PATH=B:\Penetration\漏扫\二开dddd\dddd64.exe"
set "URL_LIST=B:\Penetration\漏扫\ur.txt"  // 修正为 ur.txt
:: 检查afrog路径
echo 正在检查afrog路径...
if not exist "%AFROG_PATH%" (
    echo 错误：找不到 afrog.exe，请检查路径！
    pause
    exit /b 1
)
:: 检查nuclei路径
echo 正在检查nuclei路径...
if not exist "%NUCLEI_PATH%" (
    echo 错误：找不到 nuclei.exe，请检查路径！
    pause
    exit /b 1
)
:: 检查ddddd路径
echo 正在检查dddd路径...
if not exist "%DDDD_PATH%" (
    echo 错误：找不到 dddd64.exe，请检查路径！
    pause
    exit /b 1
)
:: 检查URL列表文件
echo 正在检查URL列表文件...
if not exist "%URL_LIST%" (
    echo 错误：找不到 ur.txt，请检查路径！  // 修正提示信息
    pause
    exit /b 1
)
echo 所有路径验证通过，正在启动扫描...
echo.
:: 启动afrog扫描
start "Afrog漏洞扫描" cmd /k "%AFROG_PATH%  -T "%URL_LIST%""
:: 启动nuclei扫描
start "Nuclei漏洞扫描" cmd /k "%NUCLEI_PATH%  -list "%URL_LIST%""
:: 启动dddd扫描
start "DDDD漏洞扫描" cmd /k "%DDDD_PATH% -t "%URL_LIST%""
echo 已在独立窗口中启动三个扫描工具。
echo.
echo 按任意键退出此启动器窗口...
pause
exit /b 0
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWmToRzH3s9nbJZO7Dx0PicZpW1oibtY7dfwFiaQCpsAtWKC0qCZPhhtGia0fUlJ3fSrEQ18bKIDL7ibestespmxe5LD8GGTDwW1WUw/640?wx_fmt=png&from=appmsg "")  
### 利用工具  
  
这块其实就是一个储备的过程,多收集一些工具,在子域名/  
IP  
全端口进行漏扫后,等待过程就可以会去看看无影会扫出哪些资产和高危框架指纹,后续就是利用去打的过程了,经过这几次攻防发现最多的还是帆软、  
shiro  
、  
Jeecg  
、  
Strue  
、  
xxljob  
,看到指纹就验证一下,但是工具也会有误报情况,系统改一下接口名称工具就会识别不出。漏洞复现能力也非常重要  
  
tips：综合漏洞利用工具也可以在内部知识星球获取  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVsGIia8ftdWmWRJJaIt8qpzCHT2NM1nOdg6ofE403cXUd0lXa8NSKS7r2h0uQthaU5nGOqDiaawQDVIVyxMbdwb05YgCNfjHb30/640?wx_fmt=png&from=appmsg "")  
  
各种  
oanday  
虽然在好多都被修了,但是弱口令依然是重灾区,还有测试的账号姓名,公司名简写的口令,接口出现姓名就可以组合社工字典简写出账号去爆破,利用  
JS  
接口找到工号规律进入后台最好,这也是我的管用手法,无影的辅助能可以很好的组成字典,并且建议师傅们读一读L@2uR1te师傅这篇文章,相信您会被爆破有不一样的思考。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAU8TCowruwd3PiadnAXicMnOyniaBicHib6mib5aHzAfvZPqpO958KxWoph1Cic0FicorcA4C7zfhFicG0osQOKsmJibkftB0ICfBUEg6sKc/640?wx_fmt=png&from=appmsg "")  
## 案例  
### 静态官网  
  
遇到只有官网资产的公司也不要放过,像这种一看就是远古时期的网站往往这种一级目录下就会放置管理员后台,基本找到后台没有注入也有弱口令,然后看看头像能不能上传  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVvib1wBM2OtgkLwF9IKguOeKmLcyibsbrRIKFxn1zFpoNwhePo6Nqjr0l1SIzzla2W4qjViaibBibF1GeJUq5h2micGYEpHeicYU41l8/640?wx_fmt=png&from=appmsg "")  
  
目录的话一般可以手动拼几个常见的,没有就上  
dirserch  
跑一遍,并且如果想要拉开差异化就需要自己多整合一些自己的目录或密码等字典,很早之前我一直是用  
dirseach  
默认的字典,但是发现相同的资产我朋友可以扫到目录但是我却不可以,发现这一点后每次遇到什么接口好的字典就会拓展到自己的字典内。核心就是整合+去重,除去很高难度的漏洞，大部分漏洞其实换一个师傅也能挖到,只是看谁先发现这个点  
```
admin
login
manage
backend
index
.....
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUEWhWkzx5p3XOWGiaB6DFvk5shlogOkDNPP0qlibGZHzeH3ay82t4ghdpYybTTm1DlicyibdEnicq017OlxUBU0eBO0KPRT63JLzpk/640?wx_fmt=png&from=appmsg "")  
  
直接就是注入起手,构造  
payload  
直接出,懒的话  
sqlmap  
梭哈  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUxv63kpb8uicvlzF9RDHsmoibVFNpV2q41kdniaP3QFfibqJrlia3qabfHd4JtaOUwz1bACz35Dbmu3GYOR4zR8FQwffvWFH3snWbY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXkzazqIwkwjMicv4zgAkjPeAicRXL1wsgWsW7al60ahrjy5IY4QZqdve19drjanZAMKPLib88BVjE7dOPuLHbiawnqkas8dzfibCnA/640?wx_fmt=png&from=appmsg "")  
  
用户名注入那么闭合了肯定是可以构造万能密码  
```
账号:admin’# 或者 1'or+1#
密码随意
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVEE9SxCZOr0aLrULdRWWXQSPPCUucYJOOLpP7JqBThPcpHfOfvtXtk3UE7Hl1NYQXlBibcCib46lWIdt6nKWH8Rkw9SkDJC3ZM0/640?wx_fmt=png&from=appmsg "")  
  
  
遇到过系统多用户使用情况了；扫描全端口资产定位  
xxx  
系统,万能密码可以进入后台,但却不是管理员的,只有个人数据,  
admin  
账户同样如此,那么判断到系统是被多人使用的,需要定位要管理员具体账户名称才可以使用万能密码登录  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVIEEU9GojIPvpSRwOBt9xCF8D9FYgM6XOayhYa8ZmElL1VH9JU0iaF94icibsZHkibtINDd9E2tfXBzbicXupZnscK93MnQaabKZ6U/640?wx_fmt=png&from=appmsg "")  
  
进入猜测的过程将常见的用户名进行爆破，最终确定  
test  
用户为管理员构造万能密码进入后台  
```
roo
systemadmin
postgres
administrator
sysadmin
Ceshi
......
test'#
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWhOgNAlbIaxt77CJ4ypgoBeZnT5iaRrX0SS2FloyRUqbtv5l4icW8LGibtRHSMNbIt172DvNyy5fR05QjicHqfMrF5eK7riccjrCZg/640?wx_fmt=png&from=appmsg "")  
### druid利用  
  
druid  
利用方式大部分都是关注一下  
session  
监控没有过期看能不能进入后台和数据库连接弱口令问题,但如果二者都没有的话可以观察一下  
UR  
监控栏,面板会带出所有访问的接口请求,可以和  
swaagger  
一样复制作为接口去爆破  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAVfP5r9GIENfjA9yhfQ7pGf5jVibfulXDf4ET1oHylPl3IjKSkiackHDkUTCu52iaic5WDWLyaG3KmURHwTXwf99xO1QstBUletSeA/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAUvqrMPD1qMv2gUhz9icbf5qB6zcB5e5W80geUQ5ict8y192iarUKp0OU7PQckOKLt9harblTWcPQibDAiazm6RHQN6pzW8DjygPrv8/640?wx_fmt=png&from=appmsg "")  
  
利用这个操作发现一处接口可以下载全部的  
SQL  
文件虽然没有三要素信息但是攻防也是收取公司内部业务数据的,  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAVE3YL0dO1icu03lBoLFXiahftpJ8Xic2UPL4x9OVd1AdAjozKSS5dea6qToauoTrOQujK2zP2fkfK7fTZWticwzN8lFDD4HP1w3GM/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWDkTbpm6ricEKDWqU1H5ynnrCo5ibFjW1L1nsXG0xUpA3GjZ7y0xLSvI5gquleBDT0yEDiaw3pYaia0BFeDicbyWFia7PmleGKnCXkE/640?wx_fmt=png&from=appmsg "")  
### web反推小程序  
  
弱口令测试发现有滑块验证,那么我自然是懒得上插件绕过验证,随即开始测试接口  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAUKJoLOMQbuT47ezicGicnfrBz4ia03dyibm4hkMNtF87IOaAfuicJlKWQnuhFicpnKMBHqTHJzuwcPKXA1BI11JDB1jNX5FJ5D0j3D8/640?wx_fmt=png&from=appmsg "")  
  
报告截图不完整,最终在一处  
JS  
发现硬编码密码  
admin123  
登录后台,获取大量学生信息,浏览功能点没有发现可以命令执行的地方为洗衣管理后台,学生洗衣大部分是用小程序控制的,从首页的小程序交易也可以发现  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAV6qX06Sa0WCvia8mBROibPxibvSd18Rfex0PyNX02xQYsgJADDbIECZhloR8vP1qBX3wexhDVdcT6JQFY9ico4WMj0EI0ianxJhCs8/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXunT8Gl0B6P1WibyNR5rEJWoQRIJlfPkZ3x0Ul7lK0pqLW8OCShNxY49Jg7AsXXDvtib4HecNia8EbO0aiahMoHBuGGzDB45snNic4/640?wx_fmt=png&from=appmsg "")  
  
那么直接快进到小程序渗透,根据系统名称定位小程序随便找一个学生账号登录  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAUibIzB3ATKLiathenLsm566oqee0LjJBSDM3vIk6jcmHlg96pKBSaE1xlOCEs3VicXHboQKZqTE6sAPf8bYkJnuvPtofOYNXN3zU/640?wx_fmt=png&from=appmsg "")  
  
  
依旧无鉴权情况,发现多个越权漏洞和信息泄露包括涉及金额交易的0元购,贴几张截图证明危害  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAXbOlwbznYKNKsg79yQ5RK1IPicOQ32WotcOoAFLkgvLSeUc8gDtqQUpTDsOMYA7ibibk3PO49zZtG3jK8AS5IR45nic1JZibuC42gw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAU6Fa5a9UeWwENo8j9KGCqYxjKytnZvFtpvctJSXtwHBAWKm5iank6Ae1Mp7F29jjdNpWFYWDdYSiaq3rQplytJR9wjFvfcVsyd0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAVKLmDokjakfU21smRiavabcgsM9WPtibYVursXr1mwC1gy6Gm5CSXR2miasNUPibFP72RgFatVSsjonef3XbwwjXicCgkv0kbHOicqg/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUYnBlLFACDQv60gJBa3xNa2KgL4fIoP3CtZ2dvqicicGfO48lMBM5iczmCM0YibugkWYibZoYIrpbtC9DHYCnJM7gM5zydR8Y5iaxgo/640?wx_fmt=png&from=appmsg "")  
### 民生业务  
  
在攻防社区发过一篇文章说民生业务信息收集小思路,在医院、水务等需要账号进行测试情况下,如何收集到有效的账号就变得至关重要,我也是利用这一思路  
```
https://forum.butian.net/share/4619
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXBmMXwdnEG5M2fw32g15jba2sWILpsgiaT6MsZ0OzibE8zvy7BKicd87DNDnfGnpHfDZXASEe1vgKHzcSoRlErxHNr9XicayV0mw0/640?wx_fmt=png&from=appmsg "")  
  
抖音小红书均可,收集到有效挂号号码、身份信息等参数,方便我们测试信息泄露或者得到有效的账号,这里也是全院信息泄露  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVtKJ6Pnec8ePMFBXtTic0cyU3hia5jYJ9rzEXiaqbzlnG1yeHZSzTZJy3ebokYHyMJUfwrJicSxHdmAzP48RsX1qtFWTfcIWYgafQ/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWaAkPzFHY5RSicWuwPndN5x5ZzzQ275xnFAicXjDvq0vIUIby1raib2lUQEmr9IicqOtl13M5X6tXz0HMhShOqkXIvR3eGvnODicuA/640?wx_fmt=png&from=appmsg "")  
### 帆软  
  
漏扫到存在帆软  
channel  
反序列后工具梭哈正常上了冰蝎马,再上了  
vshell  
进行流量代出,晚上还是在正常打内网,到了第二天白天发现  
vshell  
权限掉了,但是冰蝎还可以连接,排查一番机器变成了不出网,没遇到这情况的还在准备上传其他工具做正向代理,队长已经给我指出让我注入  
suo5  
内存马了,其实细想是很基础的问题但没接触过一时间想不到这个点,记得一次队友进入内网但是死活上不了  
C2  
,因为常见内网机器无非就是  
Linux  
windows  
 排查各种问杀软,,不出、系统内核最终发现机器竟然是  
ma  
 利用cs插件生成  
mac  
木马上线,经验还是很重要的  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVxPyhUickfAWGdaJt6qXUxjZvOp3rcG0f64TaUibCWYuMvPolUJ8icjzDemzrOt4nmg59AWJnjEXBYTwKklqJqFIDob5VEICCnrY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUQrLHBuNTIrWYefrshaUnorI328PyXZ79OQfR2tA8IicedYgXbNiahzdO9I9oEkKClYiaTFn3tCibteX6w0VAPzStUFV2XsP1iaxp8/640?wx_fmt=png&from=appmsg "")  
  
利用  
suo5  
隧道搭建正向连接即可  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXDogBOEohyKsqEXic6P8K37vicK18icFRiaOR1rZ4qwIYtWrFoPZHe0swsrJuvibLlibHiaQsmtdVSNQXMzyY639g6Ckync6bwvbGw5s/640?wx_fmt=png&from=appmsg "")  
  
查看进程无杀软后目标上传  
Fscn  
梭哈,扫描常见三个内网  
C  
段,还可以扩大到  
B  
、  
A  
段,还可以利用  
netspy  
进一步探测内网可达网段,我会将信息导出到  
txt  
 保存到本地用  
FscanParser  
打开  
```
fscan.exe -h 10.200.1.1/8  -o 10.txt
fscan.exe -h 192.200.1.1/8 -o  192.txt
fscan.exe -h 172.200.1.1/8 -o   172.txt
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAUA0xguaMrxibb7CNetFcHlvciazUwY0UFlPv9SvJZPA9ibBreuh0Q4JAvvr6WQr7abSCKgKCXNtmFxiaZLIaibenLcNq8G44mFia7Zo/640?wx_fmt=png&from=appmsg "")  
  
Proxifier  
全局代理流量后就是依次访问资产了,刷刷  
FTP  
、打印机、还有各种未授权弱口令的过程  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUKdldia4W0cwl4PQHkqNBnyLXxcrDAv86wtic9Mt1iaw6icbSVGnd6krUc3oicwpUnhqxjSPt6hhXM6xcDDicEic0YYpZfDliaMmXK7lg/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAWavKddw1JfPcP78V4p4nuVXW6V8D4rtAdOhofmc19DtVbG0LYEGgnwuVplwWsywHK2uJSgzmTcIQYxOtlKiaASeMwfYvDlibFZw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAXOURIgeR8b4icQJZ7yFdFIF4bloyHAG0U3qK7ds6BeVl54eicA5VmyNaCOLu0S1Jia6pONImYQI368RGb5FBkhiajU1mpdAj89JwQ/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAVhWjS4jggicjaMdFTSTq7uIqUbsFFibfUiaial1EdYxUCzDkGZiclcu9tQ9gI9wfeKLRnpgJv1nyC64ibpHfgqWfaD3icHrl6oo1tMto/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibrevicNauKAUVOA1yhtLOFEzwC2ic3K9U8kuCnuWtMGOgniboZ8QLMqWnyoGbn8xlHTSeGDZN12zHMPm9dTibQibF8gcrG9LX7OUYo1lH6KjuXMs/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAVEnPeMOSHRycw1dZCacDibTibzRMd2bQTphs7k9bpbdRo3zm6XURPjEhcJ0uCCkKdUL2XFwhwZbBxCy8r5uzFPlBk7ZTRrfKLEo/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ibrevicNauKAWiaHBlRAY9S5tjGGYyicKJeBXuMFmHiacqL0aNbuI46DJv6bVz50o4goesBk3XXCicS1W3KAFtIlP3YRWbLWTpWTmUFruDy6ibfDY0/640?wx_fmt=png&from=appmsg "")  
  
  
## 0x03 总结护网演练可不是 SRC 慢慢挖漏洞，几百家目标堆在面前，拼的就是手速与自动化。死磕单个目标基本等于陪跑，全靠工具链疯狂打点。ENScan 扒控股资产、无影做测绘，Nuclei、Afrog、DDDD 轮番上阵，脚本一键批量开扫。工具终究只是辅助，会踩坑也会捡漏，Druid、帆软、弱口令遍地开花，运气和实战经验缺一不可，红队打靶，三分靠工具七分靠积累！🔥喜欢这类文章或挖掘SRC技巧文章师傅可以点赞转发支持一下谢谢！内部星球VIP介绍V1.5（更多未公开挖洞技术欢迎加入星球）如果你想学习更多另类渗透SRC/AI挖洞技术/攻防/免杀/应急溯源/赏金赚取/工作内推，欢迎加入我们内部星球可获得内部工具字典和享受内部资源/内部群🔥🚀1.每周更新1day/0day漏洞刷分上分，目前已更新至13794+;🧰2.包含网上的各种付费工具/各种Burp漏洞检测插件/fuzz字典/SRC挖洞SKILL等等;3.Fofa/Hunter/Ctfshow/360Quake/Shadon/零零信安/Zoomeye各种账号VIP会员共享等等;🎥5.最新SRC挖洞文库/红队/代审/免杀/逆向视频资源等等;🧪6.内部自动化漏扫赚赏金捡洞工具，免杀CS/Webshell工具等等;💡7.漏洞报告文库、AI挖洞技巧、共享SRC漏洞报告学习挖洞技巧；🎯6.最新0Day1Day漏洞POC/EXP分享地址（同步更新）;https://t.zsxq.com/jVcxV(全网最新最完整的漏洞库)🔥7.详情直接点击下方链接进入了解，后台回复" 星球 "获取优惠先到先得！后续资源会更丰富在加入还是低价！（即将涨价）以上仅介绍部分内容还没完！点击下方地址全面了解👇🏻👉点击了解加入-->>2026内部VIP星球福利介绍V1.5版本-1day/0day漏洞库及内部资源更新结尾免责声明获取方法回复“app" 获取  app渗透和app抓包教程回复“渗透字典" 获取 一些字典已重新划分处理（需要内部专属fuzz字典可加入星球获取，内部字典多年积累整理好用！持续整理中！）回复“书籍" 获取 网络安全相关经典书籍电子版pdf最后必看    文章中的案例或工具仅面向合法授权的企业安全建设行为，如您需要测试内容的可用性，请自行搭建靶机环境，勿用于非法行为。如用于其他用途，由使用者承担全部法律及连带责任，与作者和本公众号无关。本项目所有收录的poc均为漏洞的理论判断，不存在漏洞利用过程，不会对目标发起真实攻击和漏洞利用。文中所涉及的技术、思路和工具仅供以安全为目的的学习交流使用。如您在使用本工具或阅读文章的过程中存在任何非法行为，您需自行承担相应后果，我们将不承担任何法律及连带责任。本工具或文章或来源于网络，若有侵权请联系作者删除，请在24小时内删除，请勿用于商业行为，自行查验是否具有后门，切勿相信软件内的广告！往期推荐1.内部VIP知识星球福利介绍V1.5版本0day推送2.最新Nessus2026.2.9版本下载3.最新BurpSuite2026.1.1专业版下载4.最新xray1.9.11高级版下载Windows/Linux5.最新HCL AppScan_Standard_10.9.1下载渗透安全HackTwo微信号：关注公众号获取后台回复星球加入：知识星球扫码关注 了解更多上一篇文章：Nacos配置文件攻防思路总结|揭秘Nacos被低估的攻击面喜欢的师傅可以点赞转发支持一下  
  
  
  
