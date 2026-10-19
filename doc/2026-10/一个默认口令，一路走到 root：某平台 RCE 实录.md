#  一个默认口令，一路走到 root：某平台 RCE 实录  
原创 炫彩之星
                        炫彩之星  菜鸟学安全   2026-10-19 00:00  
  
　　很多人看到“默认口令”四个字，下意识反应是：这也算漏洞？上线前改掉不就行了。可在真实的攻防里，一个没改的默认口令，往往不是收尾的小问题，而是整条攻击链的起点。这次的目标是一套xx平台，业务上关联xx、案件与个人信息；而我们从登录框走到 root shell，中间没有零日，靠的也不是什么高深技巧——真正起作用的，是几个本该被拦住的配置问题。  
  
　　在某地的一次攻防项目中，我们发现某xx平台（xx.xx.xx.xx:28806）存在默认口令→任意文件写→root 远程命令执行、refresh-token 绕过账户锁定等 2 项高危漏洞。目标服务器已被完全控制，攻击者取得 root 权限。  
  
       本次对目标地区的资产开展渗透测试，共发现高危漏洞 2 个，漏洞情况如下表：  
<table><tbody><tr style="height:18.0000pt;page-break-inside:avoid;"><td data-colwidth="58" width="58" valign="center" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(164, 164, 164);"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;page-break-after:avoid;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:10.5000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">序号</span></font></span></p></td><td data-colwidth="241" width="241" valign="center" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(164, 164, 164);"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;page-break-after:avoid;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:10.5000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">系统名称</span></font></span></p></td><td data-colwidth="125" width="125" valign="center" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(164, 164, 164);"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;page-break-after:avoid;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:10.5000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">漏洞名称</span></font></span></p></td><td data-colwidth="62" width="62" valign="center" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(164, 164, 164);"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;page-break-after:avoid;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:10.5000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">漏洞数量</span></font></span></p></td><td data-colwidth="208" width="208" valign="center" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(164, 164, 164);"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;page-break-after:avoid;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:10.5000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">漏洞影响</span></font></span></p></td><td data-colwidth="69" width="69" valign="center" style="padding: 0pt 5.4pt;border-width: 1pt;border-style: solid;border-color: windowtext;background: rgb(164, 164, 164);"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;page-break-after:avoid;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:10.5000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">漏洞级别</span></font></span></p></td></tr><tr style="height:25.0000pt;page-break-inside:avoid;"><td data-colwidth="58" width="58" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:Calibri;mso-fareast-font-family:宋体;mso-bidi-font-family:&#39;Times New Roman&#39;;font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">1</span></span></p></td><td data-colwidth="241" width="241" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">https://xx.xx.xx.xx:28806/api/admin-api/system/auth/login → file-config/create → file/upload</span></font></span></p><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">系统：某</span></font><font face="仿宋"><span leaf="">xx平台</span></font></span></p></td><td data-colwidth="125" width="125" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">root RCE</span></font></span></p></td><td data-colwidth="62" width="62" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">1</span></font></span></p></td><td data-colwidth="208" width="208" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="margin-top:0.0000pt;margin-right:0.0000pt;margin-bottom:0.0000pt;margin-left:0.0000pt;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">默认口令直达</span></font><span leaf=""> root 完全控制，并据此导出全量凭据，是后续横向攻击的起点。</span></span></p></td><td data-colwidth="69" width="69" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">高危</span></font></span></p></td></tr><tr style="height:25.0000pt;page-break-inside:avoid;"><td data-colwidth="58" width="58" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:Calibri;mso-fareast-font-family:宋体;mso-bidi-font-family:&#39;Times New Roman&#39;;font-size:10.5000pt;mso-font-kerning:1.0000pt;"><span leaf="">2</span></span></p></td><td data-colwidth="241" width="241" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">https://xx.xx.xx.xx:28806/api/admin-api/system/auth/refresh-token</span></font></span></p><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">系统：某</span></font><font face="仿宋"><span leaf="">xx平台</span></font></span></p></td><td data-colwidth="125" width="125" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">refresh-token 绕过账户锁定</span></font></span></p></td><td data-colwidth="62" width="62" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">1</span></font></span></p></td><td data-colwidth="208" width="208" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="margin-top:0.0000pt;margin-right:0.0000pt;margin-bottom:0.0000pt;margin-left:0.0000pt;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">账号锁定后仍可无限续签超管会话，使锁定与应急响应失效。</span></font></span></p></td><td data-colwidth="69" width="69" valign="center" style="padding: 0pt 5.4pt;border-width: medium 1pt 1pt;border-style: none solid solid;border-color: currentcolor windowtext windowtext;"><p style="text-align:center;text-indent:0.0000pt;mso-char-indent-count:0.0000;mso-pagination:widow-orphan;"><span style="font-family:仿宋;color:rgb(0,0,0);font-size:12.0000pt;mso-font-kerning:0.0000pt;"><font face="仿宋"><span leaf="">高危</span></font></span></p></td></tr></tbody></table>  
## 任意文件写 → root RCE  
### 漏洞描述  
  
　　资产详情：  
https://xx.xx.xx.xx:28806  
  
　　系统名称：某xx平台  
  
　　漏洞地址：① POST   
https://xx.xx.xx.xx:28806/api/admin-api/system/auth/login  
  
　　② file-config/create → file/upload?path=etc/cron.d/artexrce  
  
　　漏洞说明：yudao 框架默认超管口令 admin/adxx3 未修改，攻击者无需任何前置认证即可登录并取得超管会话；infra/file-config 可新建 basePath=/ 的 Local 存储，infra/file/upload 的 path 参数可写任意绝对路径（以 root 身份落盘）。  
  
　　危害：攻击者可完全控制某xx平台及其所在服务器。该主机同时承载 Go 业务平台、Spring 业务接口、MySQL、PG、Redis 与 FTP 等多套服务，一旦失陷，业务数据、数据库凭据与主机权限将一并暴露。  
  
       https://xx.xx.xx.xx:28806/#/  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gr6fLkqy0IakRLawp0KrO6BSdJvt0UcZ55zMic6nbFxec4j1jPCN8ibjOxNGO98zFamuJBiaQzmERS5eFBe7aUHOhknA1GUuyRWAU/640?wx_fmt=png&from=appmsg "")  
  
         默认超管口令 admin/adxx3登录。  
  
        https://xx.xx.xx.xx:28806/#/pages/login/index?type=2  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/oFamj4ye1gqcwbofDDTegZcvGdrHOWLlesIRq1HTVOXiaBBAeRffyvxLWDKNGOW5DibibD1n1dZ8Md3UTHvoBeHN0FGicm35mfmsS9eNQhoJkAA/640?wx_fmt=png&from=appmsg "")  
  
        成功登录后台页面。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqpFP5Oibzf2IneduiaCicZ5AbjAYic3AILS0GpAwLibcbJAZ39k7lRVDWGpr9b1Mca2gK7BFcxQYHDzs0cxn0lL6KtL7fFltJp2iaPY/640?wx_fmt=png&from=appmsg "")  
  
　　复现过程（完整利用链）：  
```
```  
```
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gpTppqtlXoCu5QcuL2wHd94VSuC1icm1ibIicpxDLPyEnuSHYvlXopWkia8u7jIXSibaO3fyrSXgge0eeJx9JThiaNbBSBibVia7jNkm9M/640?wx_fmt=png&from=appmsg "")  
```
```  
```
```  
```
```  
```
```  
```
```  
```
```  
```
```  
```
```  
```
```  
```
```  
```
```  
  
成功获取 root 反弹 Shell。  
```
```  
```
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/oFamj4ye1gqkY9phDkom1GstPugXWvqVtdeTR8cJKvjSCoOXcdHghORzcSyYjxpboiazrAdt1mtPusSQ9tvXmoibZ4nyib5RvPKM1GEtdBezqM/640?wx_fmt=png&from=appmsg "")  
```
```  
```
```  
## refresh-token 绕过账户锁定  
### 漏洞描述  
  
　　资产详情：  
https://xx.xx.xx.xx:28806  
  
　　系统名称：某xx平台  
  
　　漏洞地址：POST   
https://xx.xx.xx.xx:28806/api/admin-api/system/auth/refresh-token?refreshToken=  
…  
  
　　漏洞说明：账号锁定仅拦截「密码登录」，未吊销已签发的 refresh token，导致锁定无法真正中断攻击者的会话。  
  
　　危害：yudao 账户锁定机制失效，攻击者可在账号被锁定后持续续签超管会话，使封禁与应急响应措施形同虚设。  
  
　　复现过程（完整利用链）：  
```
```  
```
```  
```
```  
## 写在最后  
  
　　复盘整条链路，真正致命的地方并不是某个“高级漏洞”，而是几个再普通不过的配置问题被串在了一起：默认口令没有改，超管会话直接到手；文件配置允许把 basePath 指向根目录；上传接口的 path 没有做归一化，应用又以 root 身份运行。这些环节里只要有一个被拦住，root shell 都不会出现。  
  
　　另一个漏洞更值得警惕：账号锁定只拦住了密码登录，却没有吊销已经签发的 refresh token。对运营方来说，“锁定”并不等于“断线”；对攻击者来说，只要拿到过有效令牌，锁定就只是一句提示。修复方向其实不复杂：清掉默认口令、应用降权运行、上传路径做根目录限制，并在锁定时同步吊销 refresh token。  
  
　　渗透测试的意义，不是证明我们能打进去，而是证明一条本不该存在的路确实存在。把这条路上的每一环补上，比事后追着一个 root shell 更有价值。  
  
