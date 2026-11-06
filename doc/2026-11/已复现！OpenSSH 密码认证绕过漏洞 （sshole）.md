#  已复现！OpenSSH 密码认证绕过漏洞 （*sshole）  
微步情报局
                    微步情报局  微步在线研究响应中心   2026-11-06 00:30  
  
漏洞概况  
  
  
11月5日，OpenSSH官方紧急发布10.6p2版本，修复并披露了一处sshd密码认证绕过漏洞（主干修复Commit：4353fdc43ef646c43a8dda7025cc1a898caf1482）。  
  
经微步情报局研判，此漏洞影响 7.3p1 至 10.6p1 的Portable OpenSSH版本，  
但触发条件依赖特  
定运行环  
境和账户配置，仅在Linux系统底层使用 musl libc（如Alpine Linux等容器或发行版）、sshd编译或运行时未启用PAM支持且开启密码认证（PasswordAuthentication yes）时才受影响，使用glibc或启用PAM的常规主流Linux服务器环境无需恐慌。  
  
  
利用条件：  
  
系统配置  
  
    1. 目标系统是 Linux，且 C 库为 musl（例如Alpine）  
  
    2. 不启用PAM（Alpine 默认软件包 openssh-server 未启用 PAM）  
  
    3. PasswordAuthentication yes。  
  
    4. MaxAuthTries > 1 （OpenSSH 默认 MaxAuthTries 6）  
  
账户配置  
  
    5.有一个账户，口令字段是 *（没设口令，不是锁定）  
  
    6.另有一个账户，口令字段是 $5$ 或 $6$ 哈希（设过口令）  
  
    这两行不必挨着，也没有顺序要求，存在即可  
  
攻击者  
  
    7.能连到这台机器的 SSH，并知道那个 * 账户的用户名  
  
    8.对该账户发 257–1024 字节口令。一次连接即可（做两次口令认证）  
  
  
用户可通过如下方案快速自查是否满足受影响前置条件：  
1. 检查版本与C库环境：执行 sshd -V 确认版本处于 7.3p1 ~ 10.6p1 之间，并执行 ldd $(which sshd)，若输出中包含 musl 且未链接 libpam.so（或在链接了 libpam.so 时配置了 UsePAM no），则满足环境前置条件；  
  
1. 检查密码认证是否开启：执行 sshd -T | grep -i "passwordauthentication"（并检查 /etc/ssh/sshd_config 中是否存在 Match 块覆盖配置），若结果为 passwordauthentication yes，则存在受影响风险。  
  
建议受影响用户尽快将OpenSSH升级至 10.6p2 版本。  
  
（完整漏洞情报请查阅https://x.threatbook.com/v5/vul/XVE-2026-91288）  
  
漏洞影响范围  
  
<table><tbody><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">产品名称</span></section></td><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;vertical-align: top;"><section><span leaf="">Portable OpenSSH（Linux / musl libc 环境）</span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">受影响版本</span></section></td><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;vertical-align: top;"><section><span leaf=""> 7.3p1 至 10.6p1</span></section></td></tr><tr><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;vertical-align: top;font-weight: bold;background-color: rgb(248, 249, 250);"><section><span leaf="">有无修复补丁</span></section></td><td style="border: 1px solid rgb(221, 221, 221);padding: 12px;vertical-align: top;"><section><span leaf="">有</span></section></td></tr></tbody></table>  
漏洞复现  
  
  
微步情报局已在 Alpine Linux 3.22.7, OpenSSH 10.6p1下复现成功。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/T4OSm0sXdEPeaOI5hh4URzfAIlNkrTgITu4N0vmVWNN980z4AhApibXflfjTnfrN1c93WiaybnSB49PB0OsfHEskv22O1liagWiamibxTMbTP4Ns/640?wx_fmt=png&from=appmsg "")  
  
修复方案  
  
### 官方修复方案  
  
官方已发布修复方案，请访问链接下载：  
  
https://www.openssh.org/txt/release-10.6p2  
  
主干修复 Commit：4353fdc43ef646c43a8dda7025cc1a898caf1482  
### 临时缓解措施  
  
在 /etc/ssh/sshd_config 中设置 PasswordAuthentication no（同时确保无 Match 块局部开启）以禁用密码认证、改用公钥认证，并重启sshd服务即可完全阻断。  
微步产品支撑  
  
  
微步漏洞情报于  
 2026-11-05   
收录该漏洞。  
  
微步下一代威胁情报平台NGTIP及X情报中心已于漏洞收录时向漏洞订阅用户推送该漏洞情报，并将持续推送后续更新；对于已经录入资产的用户，支持实时自动化排查受影响资产。  
  
